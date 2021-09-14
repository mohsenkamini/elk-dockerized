<div dir="rtl">

  
# راه اندازی و کانفیگ filebeat
 دیزاین کانتینر
 filebeat
 برای دریافت لاگ های هاست میزبان خود.
 در ادامه کار و کانفیگ ها دیاگرام توضیح داده خواهد  شد.
  ![filebeat diagram](https://user-images.githubusercontent.com/77579794/133249252-cce2e73b-a2b4-4533-96a8-9464f15504c4.png)
مسیر فایل های کانفیگ و راه اندازی مربوطه به filebeat در مسیر `/elk-dockerized/docker-compose/filebeat`
 قرار دارد و هر مسیری در ادامه گفته میشود relative به این مسیر میباشد.
 
 برای هر host ای که قصد دارید لاگ های سیستمی آنرا دریافت کنید در ابتدا این ریپو را با یوزر روت روی آن host دریافت کنید و به مسیری که در بالا گفته شده بروید.
 
 در ابتدا فایل compose مربوط به فایل بیت را مورد بررسی قرار میدهیم و بعد به فایل کانفیگ آن میپردازیم.
 
 برای ایجاد دسترسی های بیشتر برای یوزر filebeat داخل کانتینر, ایمیج مربوط به این سرویس در حد خیلی کمی شخصی سازی شده و در هنگام اجرای فایل کامپوز ابتدا ایمیج شخصی سازی شده آماده خواهد شد. در صورت استفاده از ایمیج اصلی برخی از لاگ فایل های سیستم خارج از پرمیشن های یوزر filebeat خواهند بود و filebeat قادر به خواندن آنها نخواهد بود. مسیر داکرفایل : `filebeat-dockerfile/Dockerfile` 
 
 بخش مهم دیگر `bind-mount` تعریف شده برای دریافت لاگ های host ای ست که کانتینر بر روی آن قرار میگیرد که بدین صورت دسترسی به لاگ های سیستم میزبان برای کانتینر میسر میشود.
   <pre dir="ltr">
       volumes:
      - /var/log:/var/myhost.logs:ro
  </pre>
پس مسیر `/var/myhost.logs` در کانتینر در واقع محل بایند لاگ های سیستم میزبان میباشد. این مسیر باید در فایل کانفیگ فایلبیت تعریف شود.
 
 فایل کانفیگ اصلی فایل بیت در مسیر `data/filebeat/data01/filebeat.docker.yml` قرار دارد.
بهتره قبل از ادامه نگاهی به محتویات این فایل بیاندازیم:
  <pre dir="ltr">
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false
filebeat.modules:
  - module: system
    syslog:
      enabled: true
      var.paths: ["/var/myhost.logs/syslog*"]
    auth:
      enabled: true
      var.paths: ["/var/myhost.logs/auth.log*"]

setup.kibana.host: "https://elk.test.ir:443"
setup.kibana.protocol: "https"
setup.kibana.username: "kibana_system"
setup.kibana.password: "CHANGEME"

output.elasticsearch:
  hosts: https://elk.test.ir:9200
  username: elastic
  password: CHANGEME
 </pre>
بخش های مهم این کانفیگ:
 1. بخش `filebeat.modules`
 
همان طور که مشاهده میشود ماژول های syslog و auth لاگ های مورد نیاز خود را از مسیر های تعیین شده که لاگ های سیستم میزبان هستند دریافت, برداشت (harvest) و پراسس میکنند.
 
 2.  بخش `output.elasticsearch`
 
 در این بخش بعد از دریافت و عبور از module ها باید برای الستیک ارسال شوند که در این بخش کانفیگ و credentials مربوطه باید وارد شود.
 
 همین طور دایرکتیو های مربوطه با اطلاعات مربوط به kibana باید با اطلاعات جدید استک شما تغییر کنند.
 
 


</div>
