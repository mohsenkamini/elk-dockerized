<div dir="rtl">
  
# راه اندازی و کانفیگ logstash
  
  
  از ایجنت logstash برای دریافت لاگ کانتینر های داکر با کمک گرفتن از 
  یکی از لاگ درایور های داکر به نام `gelf` استفاده خواهیم کرد.
  
  ## دیزاین
  دیزاین و آرشیتکت کلی logstash با استفاده از نمودار زیر شرح داده میشود.
    ![logstash diagram](https://user-images.githubusercontent.com/77579794/133299933-e75b31ee-73c7-4064-a395-322d834aaf80.png)
  
  1. کانتینر هدف میتواند داخل یا خارج از نتورک داکری باشد که کانتینر logstash  حضور دارد واین مورد اصلا مهم نیست و تمامی لاگ های ارسالی 
  توسط gelf 
  ابتدا به هاست اصلی بر روی پورت 12201 فرستاده میشود و سپس توسط docker-proxy به پورت 12201 کانتینر لاگ استش میرسد.
  
  2. در نتیجه از مورد قبلی همچنین میتواند لاگ کانتینر هایی که در هاستی خارج از هاست agent  ما حضور دارند نیز ارسال کرد.
  این کانفیگ مربوط به کانفیگ `gelf output` میباشد که در ادامه در موردش توضیح داده خواهد شد.
  
  3. در نهایت لاگ های جمع آوری شده در کانتینر لاگ استش پراسس میشوند و در این مرحله برای `elasticsearch output` ارسال خواهند شد.
  
  ## کانفیگ
  
  مسیر فایل های کانفیگ و راه اندازی مربوطه به logstash در مسیر /elk-dockerized/docker-compose/logstash قرار دارد و هر مسیری در ادامه گفته میشود relative به این مسیر میباشد.
  
  دو فایل کانفیگ برای logstash در نظر گرفته شده است که به صورت bind-mount در اختیار کانتینر گذاشته خواهند شد.
  
  در مسیر `data/config/logstash.yml` کانفیگ پیکربندی logstash قرار دارد.
  کافیست فقط دایرکتیو زیر را با ادرس مربوط به هاست `elasticsearch` خود و روی پورت مربوط به elasticsearch تغییر دهید.
    <pre dir="ltr">
  monitoring.elasticsearch.hosts: [ "https://elk.mohsenkamini.ir:9200" ]
   </pre>
  
  فایل بعدی مربوط به کانفیگ `pipeline` لاگ استش میباشد که مواردی همچون:
  
  - `input` ها برای نحوه دریافت لاگ ها
  
  - `filter` ها برای پراسس کردن و عملیات روی لاگ ها
  
  - و `output` برای تعیین محل ارسال لاگ های دریافت و پراسس شده که ما از اوتپوت `elasticsearch` استفاده میکنیم.
  
  نگاهی بیاندازیم به کانفیگ هایی باید برای استک شما شخصی سازی شوند.
      <pre dir="ltr">
output {
  elasticsearch {
    hosts => ["https://elk.test.ir:9200"]
    user => "elastic"
    password => "CHANGEME"
    index => "logstash-7.14.0"
  }
}
   </pre>
   در این قسمت همان طور که مشخص شده باید credentials مربوطه را برای یوزر elastic  وارد کنید و همچنین هاست مربوط به elasticsearch را تغییر دهید.

  ## راه اندازی نهایی
  فقط کافیه در مسیر گفته شده فایل کامپوز لاگ استش را اجزا کنید.
      <pre dir="ltr">  
  docker-compose up -d
     </pre>
  می توانید روی پورت 
  `9600`
  اطلاعات مربوط به لاگ استش را ببینید و راه اندازی خود را confirm کنید.
  

<pre dir="ltr">  
http://elk.test.ir:9600/
  	
host:	"a6edf0b04dee"
version:	"7.14.0"
http_address:	"0.0.0.0:9600"
id:	"e5029ad8-53e4-4a24-88ed-802397180c09"
name:	"a6edf0b04dee"
ephemeral_id:	"59f04eda-2b0e-4c8c-b232-9508a87d2633"
status:	"green"
snapshot:	false
pipeline:	
workers:	8
batch_size:	125
batch_delay:	50
build_date:	"2021-07-29T19:43:14Z"
build_sha:	"29d52f1e490de0a97194846bbfc2aa00dce23b54"
build_snapshot:	false
</pre>
  
  همچنین در لاگ های کانتینر لاگ استش میتوانید ببینید:
 <pre dir="ltr">  
  [2021-09-14T20:17:24,900][INFO ][logstash.javapipeline    ][main] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>8, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>1000, "pipeline.sources"=>["/usr/share/logstash/pipeline/logstash.conf"], :thread=>"#<Thread:0xb130b87 run>"}
[2021-09-14T20:17:27,048][INFO ][logstash.javapipeline    ][main] Pipeline Java execution initialization time {"seconds"=>2.14}
[2021-09-14T20:17:27,145][INFO ][logstash.javapipeline    ][main] Pipeline started {"pipeline.id"=>"main"}
[2021-09-14T20:17:27,184][INFO ][logstash.inputs.gelf     ][main][f07722edb04845f529cb09f5f2766cc4d65f7b5f8264c660a306d13dd613aeea] Starting gelf listener (udp) ... {:address=>"0.0.0.0:12201"}
[2021-09-14T20:17:27,269][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
</pre>
  
  ## کانفیگ gelf برای ارسال لاگ ها به لاگ استش
  طبق چارت اول این صفحه اهمیتی ندارد که لاگ استش شما و کانتینری که لاگ ان را در استک میخواهید با هم روی یک ماشین باشند یا نباشند. فقط کافیست روی هر هاستی که هست کانفیگ لاگ درایور داکر را برای آن کانتینر تغییر دهید و یا به طور کلی لاگ درایور دیفالت آن هاست را به gelf تغییر دهید. 
  
  برای تغییر log driver دیفالت در داکر میتواند کانفیگ زیر را 
  در مسیر `/etc/docker/daemon.json` قرار دهید و به جای `ip` تعریف شده 
  اطلاعات مربوط به هاست logstash خود را وارد کنید.
  
<pre dir="ltr">  
{
  "log-driver": "gelf",
  "log-opts": {
    "gelf-address": "udp://IP-OF-YOUR-LOGSTASH-HOST:12201"
  }
}
</pre>
  و سپس
<pre dir="ltr">
  systemctl restart docker
</pre>
  از این پس هر کانتینر جدیدی که روی این هاست بسازید با این کانفیگ لاگ های خود را برای لاگ استش میفرستد.
  (کانتینر های قبلی به منوال لاگ درایور قبلی شان خواهند بود)
  
 برای کانفیگ کانتینر هایی که در حال حاضر ساخته شدن میتواند به داکیومنت های داکر مراجعه کنید. `gelf log driver`
  
  
  </div>
