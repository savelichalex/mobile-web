# mobile-web

Здравствуй, мой юный друг! Да-да, ты молодец что зашел сюда, ведь сегодня я расскажу тебе сказку о мобильном вебе.

Да, о том самом вебе на который ты все никак не можешь найти время. А меж тем мобильный трафик работает, укрепляется и правило мобайл-ферст только укрепляет свои позиции.

Скорее всего ты сидишь и думаешь, как же круто, что у меня есть мобайл версия, в которой у меня все адаптивно и все так курто.

## Извините, но это не совсем так
От того, что ты написал адаптивную верстку, некоторые проблемы не ушли.

## Давайте разберемся

И так, перед нами целый зоопарк девайсов. Что будем делать? Все верно, определим нашу ЦА! И поймем на чем ребята то сидят, котоыре нами пользуются, в моем случае это все возможные в мире девайсы. Ага.

## Проблемы с iOS

### Проблема 1 — Скейл на странице
В iOS 10, ребята немного отключили вам скейл на странице.

     <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

Я считаю это проблемой, но есть и те кто скажут что это не так. Выбор ваш.

### Проблема 2 — Scroll и overflow
Всеми любимый скролл и оверфлоу. Извините друзья, но в iOS он работает не так как вы хотели бы.


### Проблема 3 — Плавный скролл
Снова скролл, это скорее не проблема, а просто напоминание, не забудьте сделать вот такое. Позволяет сделать плавный скролл пальцем в блоке.

    -webkit-overflow-scrolling: touch;

### Проблема 4 — new Date()
Между Safari и Chrome, есть маленькая нестыковочка. Она связана с `new Date()` - в случае с Chrome вы спокойно скормите строку вида 2016-8-2, в то время как Safari будет ругаться и заставит вас доставить недостающие нули.

### Проблема 5 — Flexbox Wrap
Свойство `flex-wrap: wrap;` работает не так, как в других браузерах. Даже если указана минимальная ширина элемента через `min-width`, переноса элементов на другие строки не будет: Safari не учитывает `min-width`. Необходимо обязательно задавать `flex-basis` (либо `flex-basis: 10em;`, либо `flex: 1 0 10em;`).

### Проблема 6 — Скругление input
Браузер Safari на iOS по умолчанию скругляет все текстовые input и не реагирует на `border-radius: 0;`. Чтобы предотвратить скругление необходимо использовать `-webkit-appearance: none;`.

## Проблемы с Android

### Проблема 1 — Клавиатура в WebView
WebView и Android. К большому сожалению, есть маленькая неприятность. Если iOS делает скейл и располагает клавиатуру, так как это надо, то Android просто ее открывает и все.
Я решал эту проблему очень просто, я добавлял padding снизу моего блока. Можно поступить более логично, и скролить до нужного вам инпута. В среднем размер клавиатуры составляет 150px.

### Проблема 2 — Huawei U8860
Я по каким то причинам так и не смог ее решить, это тоже связано с версткой внутри приложения.
Есть такой девайс, Huawei U8860 / Android OS 4.0.3 и вот на нем не работает `border-radius` и это грустно.

Помимо этого, на этом же девайсе не работает linear-gradient, что тоже может сильно не радовать вас.


#Проблемы с PWA

### Проблема 1
Есть такой мета-тег который позволяет Вам, покрасить статус бар на вашем iOS, очень важная вещь если вы сохраняете свой сайт на рабочий стол. 
Но это не работает в версиях 9.0 и 9.1, в 9.2 уже все работает.

    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

### Проблема 2
Извините, но в iOS нет таких фишек из PWA. Одна из очень важных вещей, это не сервис воркеры, вы их переживете. А вот сохранения стейта приложения (состояния) в котором вы его свернули, не будет работать, это работает только в Anroid.

Плюс, я так и не понял, как в итоге пробросить гет-параметры при старте сайта с рабочего стола на iOS. Таким образом, я даже не могу трекнуть, сидит человек с рабочего стола или все же из браузера.

### Проблема 3
Сервис-воркеры, это не сербряная пуля. Извините, но придется попотеть, чтобы они работали так как вы хотите.

И так, все мы знаем что у мобайл бразера и так есть система кэширования? И что он и так все закэширует?
Так вот, ваш сервис-воркер он тоже закэширует. И файл, который устанавливает ваш сервис-воркер. И все это дело протухнет минимум через 24 часа (эта инфа про 24 часа не точная, но бороздя просторы интернета нашел именно это). И ваша новая версия кэша, просто не приедет к человеку.

Я решил эту проблему, тем что добавляю версии кэша к файла, как некий хэшик, чтобы браузер всегда скачивал актуальную версию кэша и переустанавливал в нужный момент мне сервис-воркер.

    gulp.task('generate-version-sw', () => {
      let text = file.readFileSync('dist/service-worker.js', 'utf8');
      let version = Math.random().toString(36).slice(2, 2 + Math.max(1, Math.min(10000, 10)))
          text = text.replace('v1::', version )
          file.writeFileSync('dist/service-worker-' + version + '.js',text)

      let text_upload_file = file.readFileSync('app/scripts/upload-service-worker.js', 'utf8');
          text_upload_file = text_upload_file.replace('service-worker-file','service-worker-' + version)
          file.writeFileSync('dist/scripts/upload-service-worker-' + version + '.js',text_upload_file)
          
     let text_index = file.readFileSync('dist/index.html', 'utf8');
         text_index += '<script src=scripts/upload-service-worker-' + version + '.js></script>'
         file.writeFileSync('dist/index.html',text_index)
      }
    );

Идея такова: выделяем отдельный файл который устанавливает наш сервис-воркер и сам сервис-воркер и пусть они каждый раз грузятся заново, они не такие большие и много трафика не нагонят пользователю.

### Проблема 4

Если у вас на сервере етсь какая-то логика завязанная на куках, постарайтесь не пугаться, когда вам сервис воркер, начнет ругаться в методе ```cache.addAll(requests[])```. К большому сожалению, он не прокидывает туда куки из-за чего, просто будет фейлиться и ваш сервис воркер не установится.


На этом пока что все. Подписывайтесь на твиттер https://twitter.com/gaserd123 следите за этой репой и если у Вас есть какие-то доработки, присылайте свои issue и pr.

Всем удачи и делайте мобильный веб лучше.

