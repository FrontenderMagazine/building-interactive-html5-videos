# Создание интерактивного видео на HTML5

Благодаря элементу `<video>` [HTML5][1], вставить видео на сайт теперь так же 
просто как и изображение. Учитывая то, что все ведущие браузеры поддерживают 
`<video>` ещё с 2011 года, то это ещё и наиболее надёжный способ показать людям 
ваши движущиеся картинки.

Более свежим пополнением в семействе HTML5 является элемент `<track>`. Это 
подэлемент для `<video>`, призванный упростить доступ к временной шкале видео. В 
основном он используется для добавления скрытых субтитров. Эти субтитры 
загружаются из отдельного текстового файла (файла [WebVTT][2]) и воспроизводятся 
в нижней части области отображения видео. Ян Девлин (Ian Devlin) написал на эту 
тему [отличную статью][3].

Однако, кроме как для скрытых субтитров, элемент `<track>` можно также 
использовать для любого вида взаимодействия с временной шкалой видеоролика. В 
этой статье будут рассмотрены три примера элементов для такого взаимодействия: 
метки эпизодов, миниатюры предварительного просмотра и поиск по временной шкале. 
Дочитав её до конца вы значительно лучше поймёте принцип работы элемента 
`<track>` и его API, что позволит вам создавать собственные интерактивные 
видеоролики.

## Метки эпизодов

Начнём с примера, который многим знаком благодаря DVD-дискам: метки эпизодов. 
Они позволяют зрителям быстро перейти к конкретному эпизоду. Это особенно удобно 
когда видео длинное как ролик «Синтел»: 

<iframe src="https://s3.amazonaws.com/demo.jwplayer.com/text-tracks/chapters.html" frameborder="0" height="350" width="495"></iframe>

Метки эпизодов в этом примере помещены во [внешний файл VTT][4] и загружаются на 
страницу с помощью элемента `<track>` со своего рода эпизодами. `<track>` 
загружается по умолчанию:

    <video width="480" height="204" poster="assets/sintel.jpg" controls>
      <source src="assets/sintel.mp4" type="video/mp4">
      <track src="assets/chapters.vtt" kind="chapters" default>
    </video>

Затем мы используем JavaScript чтобы загрузить подписи из текстовой дорожки, 
отформатировать их и вывести в панели управления под видео. Обратите внимание, 
что нам нужно подождать пока загрузится внешний файл VTT:

    track.addEventListener('load',function() {
        var c = video.textTracks[0].cues;
        for (var i=0; i<c.length; i++) {
          var s = document.createElement("span");
          s.innerHTML = c[i].text;
          s.setAttribute('data-start',c[i].startTime);
          s.addEventListener("click",seek);
          controlbar.appendChild(s);
        }
    });

В блоке кода выше мы добавляем 2 свойства для пунктов списка чтобы добиться 
интерактивности. Во-первых, мы прописываем атрибут `data` для хранения 
информации о начальной позиции эпизода, во-вторых, мы добавляем обработчик 
щелчка для внешней функции поиска. Эта функция будет переводить видео в 
начальную позицию эпизода. Если видео (ещё) не проигрывается, это делается так:

    function seek() {
      video.currentTime = this.getAttribute('data-start');
      if(video.paused){ video.play(); }
    };

Вот и всё! Теперь у вас есть визуальное меню эпизодов для вашего видео, 
созданное с помощью дорожки VTT. Обратите внимание что в [настоящем примере 
создания меток эпизодов][5] задействовано большее количество кода, чем было 
описано, например, код для запуска проигрывания видео после щелчка, код 
обновления позиции видео в панели эпизодов, код CSS-стилизации.

## Миниатюры предварительного просмотра

Вторым примером является замечательное средство, популяризированное 
видеосервисами Hulu и Netflix: миниатюры предварительного просмотра. При 
наведении мышки на строку управления (или перетягивании на мобильных 
устройствах), отображаются кадры предпросмотра для той позиции, к которой вы 
собираетесь перейти:

<iframe src="https://s3.amazonaws.com/demo.jwplayer.com/text-tracks/thumbs.html" frameborder="0" height="350" width="495"></iframe>

Для создания этого примера также используется [внешний файл VTT][6], загруженный 
в дорожку метаданных. Вместо текста, метки в этом VTT-файле содержат ссылки на 
[отдельное JPG-изображение][7]. Каждая метка может содержать ссылку на отдельный 
файл, однако в этом случае мы отдали предпочтение одному JPG-спрайту в целях 
сокращения времени ожидания загрузки изображения и упрощения управления 
изображениями. Метки привязаны к соответствующему участку спрайта с помощью [URI 
медиа фрагментов][8]. Пример:

    http://example.com/assets/thumbs.jpg?xywh=0,0,160,90

Вся важная логика выведения правильной миниатюры и её отображения содержится в 
обработчике события движения мыши `mousemove` для панели управления:

    controlbar.addEventListener('mousemove',function(e) {
      // сначала на основе позиции мыши рассчитываем позицию во времени ..
      var p = (e.pageX - controlbar.offsetLeft) * video.duration / 480;

      // ..затем находим соответствующую метку..
      var c = video.textTracks[0].cues;
      for (var i=0; i p) {
              break;
          };
      }

      // ..затем определяем url-адрес изображения и медиа фрагмента..
      var url =c[i].text.split('#')[0];
      var xywh = c[i].text.substr(c[i].text.indexOf("=")+1).split(',');

      // ..и наконец стилизируем перекрытие миниатюрой
      thumbnail.style.backgroundImage = 'url('+c[i].text.split('#')[0]+')';
      thumbnail.style.backgroundPosition = '-'+xywh[0]+'px -'+xywh[1]+'px';
      thumbnail.style.left = e.pageX - xywh[2]/2+'px';
      thumbnail.style.top = controlbar.offsetTop - xywh[3]+8+'px';
      thumbnail.style.width = xywh[2]+'px';
      thumbnail.style.height = xywh[3]+'px';
    });

Вот и всё. Опять же, [настоящий пример миниатюр предварительного просмотра][9] 
содержит дополнительный код. В нём предусмотрена та же логика запуска 
воспроизведения и поиска, а также отображения/скрытия миниатюры при наведении 
мыши в область панели управления и за её пределы.

## Поиск по временной шкале

В нашем последнем примере предложен ещё один способ разблокировки контента, на 
этот раз с помощью поиска внутри видео:

<iframe src="https://s3.amazonaws.com/demo.jwplayer.com/text-tracks/search.html" frameborder="0" height="400" width="495"></iframe>

В этом примере использован ранее созданный [файл с титрами VTT][10], который 
загружается в дорожку *субтитров*. Под видео и панелью управления мы выводим 
простую форму поиска:

<form>
    <input type="search">
    <button type="submit">Поиск</button>
</form>

Как в примере с миниатюрами, вся ключевая логика помещена в одну функцию. В этот 
раз это обработчик события отправки формы на обработку:

    form.addEventListener('submit',function(e) {
      // Сначала мы блокируем перезагрузку страницы и извлекаем ключевое слово/запрос..
      e.preventDefault();
      var c = video.textTracks[0].cues;
      var q = document.querySelector("input").value.toLowerCase();

      // ..затем сопоставляем все подходящие метки..
      var a = [];
      for(var j=0; j -1) {
          a.push(c[j]);
        }
      }

      // ..и выводим подходящие метки в панели управления.
      for (var i=0; i<a.length; i++) {
        var s = document.createElement("span");
        s.style.left = (a[i].startTime/video.duration*480-2)+"px";
        bar.appendChild(s);
      }
    });

Вот и в третий раз всё получилось. Как и в предыдущих случаях, [настоящий пример 
поиска по временной шкале][11] содержит дополнительный код запуска 
воспроизведения и поиска, а также сниппет для обновления текста-подсказки в 
панели управления.

## Подведём итог

На основе приведённых выше примеров вы должны получить достаточное количество 
знаний для создания собственных интерактивных видеороликов. В качестве источника 
для вдохновения, ознакомьтесь с нашими экспериментами в области [горячих точек, 
активируемых щелчком мыши][12], [интерактивных расшифровок][13] или 
[взаимодействий с временной шкалой][14].

В целом, [HTML5-элемент `<track>`][15] представляет простой в применении, 
кроссплатформенный способ добавления интерактивности в ваши видеоролики. И хотя 
вам придётся потратить некоторое время на создание VTT-файлов и интерфейсов 
такого рода, вы отметите повышение доступности ваших роликов и интереса к ним. 
Удачи!

[1]: https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Video_Text_Tracks_Format
[3]: https://hacks.mozilla.org/2014/07/adding-captions-and-subtitles-to-html5-video/
[4]: http://demo.jwplayer.com/text-tracks/assets/chapters.vtt
[5]: http://demo.jwplayer.com/text-tracks/chapters.html
[6]: http://demo.jwplayer.com/text-tracks/assets/thumbs.vtt
[7]: http://demo.jwplayer.com/text-tracks/assets/thumbs.jpg
[8]: http://www.w3.org/TR/media-frags/
[9]: http://demo.jwplayer.com/text-tracks/thumbs.html
[10]: http://demo.jwplayer.com/text-tracks/assets/captions.vtt
[11]: http://demo.jwplayer.com/text-tracks/search.html
[12]: http://www.jwplayer.com/labs/experiments/hot-spots/
[13]: http://www.jwplayer.com/labs/experiments/interactive-transcripts/
[14]: http://www.jwplayer.com/labs/experiments/timeline-interaction/
[15]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track