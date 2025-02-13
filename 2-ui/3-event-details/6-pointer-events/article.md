# Події вказівника

Події вказівника -- це сучасний спосіб обробки даних, введених за допомогою різних вказівних пристроїв, таких як миша, перо/стилус, сенсорний екран тощо.

## Коротка історія

Зробімо невеликий огляд, щоб ви зрозуміли загальну картину та місце вказівних подій серед інших типів подій.

- Давним-давно, в минулому, існували лише події миші.

    Потім набули поширення сенсорні пристрої, зокрема телефони та планшети. Щоб сценарії, які існують, працювали, вони генерували (і досі генерують) події миші. Наприклад, натискання на сенсорний екран генерує `mousedown`. Тож сенсорні пристрої добре працювали з вебсторінками.

    Але сенсорні пристрої мають більше можливостей, ніж миша. Наприклад, можна торкнутися кількох точок одночасно ("multi-touch"). Хоча події миші не мають необхідних властивостей для обробки таких мультидотиків.

- Тому були введені сенсорні події, такі як `touchstart`, `touchend`, `touchmove`, які мають властивості, специфічні для дотику (тут ми не розглядаємо їх детально, тому що події вказівника ще кращі).

    Але все-таки цього було недостатньо, оскільки є багато інших пристроїв, наприклад ручок, які мають свої особливості. Крім того, код, який прослуховує як події дотику, так і миші, досить громіздкий.

- Щоб розв’язати ці проблеми, було введено новий стандарт Pointer Events. Він забезпечує єдиний набір подій для всіх видів вказівних пристроїв.

На цей час специфікація [Pointer Events Level 2](https://www.w3.org/TR/pointerevents2/) підтримується в усіх основних браузерах, тоді як новіша версія [Pointer Events Level 3](https://w3c.github.io/pointerevents/) знаходиться в розробці і в основному сумісна з Pointer Events другого рівня.

Якщо ви не розробляєте для старих браузерів, таких як Internet Explorer 10 або Safari 12 або старішої версії, то більше немає сенсу використовувати події миші чи дотику -- ми можемо перейти на події вказівника.

Тоді ваш код буде добре працювати як із сенсорними приладами, так і з мишами.

Однак, є деякі важливі особливості, які потрібно знати, щоб правильно використовувати події вказівника та уникати несподіванок. Ми звернемо увагу на них у цій статті.

## Типи подій вказівника

Події вказівника називаються аналогічно подіям миші:

| Події вказівника | Аналогічні події миші |
|---------------|-------------|
| `pointerdown` | `mousedown` |
| `pointerup` | `mouseup` |
| `pointermove` | `mousemove` |
| `pointerover` | `mouseover` |
| `pointerout` | `mouseout` |
| `pointerenter` | `mouseenter` |
| `pointerleave` | `mouseleave` |
| `pointercancel` | - |
| `gotpointercapture` | - |
| `lostpointercapture` | - |

Як ми бачимо, для кожного `mouse<event>` є `pointer<event>`, який відіграє подібну роль. Також є 3 додаткові події вказівника, які не мають відповідного аналога `mouse...`, ми пояснимо їх незабаром.

```smart header="Заміна `mouse<event>` на `pointer<event>` у нашому коді"
Ми можемо замінити події `mouse<event>` на `pointer<event>` у нашому коді і очікувати, що вони працюватимуть нормально с мишею.

Підтримка сенсорних пристроїв також "магічним чином" покращиться. Хоча нам може знадобитися додати `touch-action: none` у деяких місцях CSS. Ми розглянемо це нижче в розділі про `pointercancel`.
```

## Властивості події вказівника

Події вказівника мають ті самі властивості, що й події миші, такі як `clientX/Y`, `target` тощо, а також деякі інші:

- `pointerId` - унікальний ідентифікатор вказівника, що спричиняє подію.

    Згенерований браузером. Дозволяє нам працювати з кількома вказівниками, такими як сенсорний екран зі стилусом і мультитач (приклади будуть далі).
- `pointerType` - тип вказівного пристрою. Має бути рядком, одним із таких: "mouse", "pen" або "touch".

    Ми можемо використовувати цю властивість, щоб по-різному реагувати на різні типи вказівників.
- `isPrimary` - `true` для основного вказівника (перший палець у мультитач).

Деякі вказівні пристрої вимірюють площу контакту та тиск, напр. для пальця на сенсорному екрані є додаткові властивості для цього:

- `width` - ширина області, де вказівник (наприклад, палець) торкається пристрою. Якщо не підтримується, напр. для миші це завжди `1`.
- `height` - висота області, де вказівник торкається пристрою. Там, де не підтримується, завжди `1`.
- `pressure` - тиск вказівника в діапазоні від 0 до 1. Для пристроїв, які не підтримують тиск, має бути `0,5` (натиснутий) або `0`.
- `tangentialPressure` - нормалізований тангенційний тиск.
- `tiltX`, `tiltY`, `twist` - специфічні властивості пера, які описують, як воно розташовується відносно поверхні.

Ці властивості не підтримуються більшістю пристроїв, тому використовуються рідко. Ви можете знайти деталі про них у [специфікації](https://w3c.github.io/pointerevents/#pointerevent-interface), якщо потрібно.

## Мультитач

Однією з речей, яку події миші повністю не підтримують, є мультитач: користувач може торкатися в кількох місцях одночасно на своєму телефоні чи планшеті або виконувати спеціальні жести.

Події вказівника дозволяють обробляти мультитач за допомогою властивостей `pointerId` та `isPrimary`.

Ось що відбувається, коли користувач торкається сенсорного екрана в одному місці, а потім кладе інший палець в інше місце:

1. При першому дотику пальцем:
    - `pointerdown` з `isPrimary=true` та певним `pointerId`.
2. Для другого пальця та інших пальців (якщо перший досі торкається):
    - `pointerdown` з `isPrimary=false` та іншим `pointerId` для кожного пальця.

Зверніть увагу: `pointerId` призначається не всьому пристрою, а кожному дотику пальця. Якщо ми використовуємо 5 пальців, щоб одночасно торкнутися екрана, у нас буде 5 подій `pointerdown`, кожна зі своїми відповідними координатами та іншим `pointerId`.

Події, пов’язані з першим пальцем, завжди мають значення `isPrimary=true`.

Ми можемо відстежувати кілька пальців, використовуючи їх `pointerId`. Коли користувач рухає, а потім прибирає палець, ми отримуємо події `pointermove` та `pointerup` з тим же `pointerId`, що й у `pointerdown`.

```online
Ось демонстрація, яка реєструє події `pointerdown` та `pointerup`:

[iframe src="multitouch" edit height=200]

Зверніть увагу: щоб побачити різницю в `pointerId/isPrimary`, ви повинні використовувати пристрій із сенсорним екраном, наприклад телефон або планшет. Для пристроїв з одним дотиком, таких як миша, завжди буде однаковий `pointerId` з `isPrimary=true` для всіх подій вказівника.
```

## Подія: pointercancel

Подія `pointercancel` спрацьовує, коли відбувається постійна взаємодія вказівника, а потім відбувається щось, що спричиняє її скасування.

Такими причинами є:
- Фізичне вимкнення обладнання вказівного пристрою.
- Зміна орієнтації пристрою (планшет повернуто).
- Рішення браузера обробляти взаємодію самостійно, вважаючи це жестом миші або дією масштабування та панорамування або чимось іншим.

Ми продемонструємо `pointercancel` на практичному прикладі, щоб побачити, як він впливає на нас.

Скажімо, ми впроваджуємо drag'n'drop для м’яча, як на початку статті <info:mouse-drag-and-drop>.

Ось хід дій користувача та відповідні події:

1) Користувач натискає на зображення, щоб почати перетягування
    - спрацьовує подія `pointerdown`
2) Потім він починає рухати вказівник (перетягуючи таким чином зображення) 
    - `pointermove` спрацьовує, можливо, кілька разів
3) І тоді відбувається сюрприз! Браузер має вбудовану підтримку drag'n'drop для зображень, яка запускається та бере на себе процес drag'n'drop, таким чином генеруючи подію `pointercancel`.
    - Тепер браузер самостійно обробляє перетягування зображення. Користувач може навіть перетягнути зображення м’яча з браузера, у свою поштову програму або файловий менеджер.
    - Для нас більше немає подій `pointermove`.

Таким чином, проблема полягає в тому, що браузер "викрадає" взаємодію: `pointercancel` запускається на початку процесу "перетягування" і події  `pointermove` більше не генеруються.

```online
Ось drag'n'drop демо з реєстрацією подій вказівника (лише `up/down`, `move` та `cancel`) у `textarea`:

[iframe src="ball" height=240 edit]
```

Ми хотіли б реалізувати drag'n'drop самостійно, тому скажімо браузеру не брати це на себе. 

**Запобігання типовій дії браузера, щоб уникнути `pointercancel`.**

Нам потрібно зробити дві речі:

1. Запобігти нативному drag'n'drop:
    - Ми можемо зробити це, встановивши `ball.ondragstart = () => false`, як описано в статті <info:mouse-drag-and-drop>.
    - Це добре працює для подій миші.
2. Для сенсорних пристроїв існують інші дії браузера, пов’язані з дотиком (крім перетягування). Щоб уникнути проблем і з ними: 
    - Запобігти їм, встановивши `#ball { touch-action: none }` у CSS
    - Тоді наш код почне працювати на сенсорних пристроях.

Після того, як ми це зробимо, події працюватимуть, як задумано, браузер не буде перехоплювати процес і не видаватиме `pointercancel`.

```online
У цьому демо додаються такі рядки:

[iframe src="ball-2" height=240 edit]

Як бачите, `pointercancel` більше немає.
```

Тепер ми можемо додати код для фактичного переміщення м’яча, і наш drag'n'drop працюватиме для пристроїв миші та сенсорних пристроїв.

## Захоплення вказівника

Захоплення вказівника є особливою подією.

Ідея дуже проста, але спочатку може здатися досить дивною, оскільки нічого подібного для будь-якого іншого типу події не існує.

Основним методом є:
- `elem.setPointerCapture(pointerId)` -- зв’язує події із заданим `pointerId` з `elem`. Після виклику всі події вказівника з однаковим `pointerId` матимуть `elem` як ціль (ніби вони відбулися на `elem`), незалежно від того, де в документі вони дійсно відбулися.

Іншими словами, `elem.setPointerCapture(pointerId)` перенацілює всі наступні події з заданим `pointerId` на `elem`.

Прив’язка усувається:
- автоматично, коли відбуваються події `pointerup` або `pointercancel`,
- автоматично, коли `elem` видаляється з документа,
- коли викликається `elem.releasePointerCapture(pointerId)`.

Для чого ж це корисно? Настав час побачити приклад із реального життя.

**Захоплення вказівника можна використовувати для спрощення взаємодії типу drag'n'drop.**

Згадаймо, як можна реалізувати користувацький слайдер, описаний в <info:mouse-drag-and-drop>.

Ми можемо зробити елемент `slider`, смугу з "повзунком" (`thumb`) всередині неї:

```html
<div class="slider">
  <div class="thumb"></div>
</div>
```

Зі стилями це виглядає так:

[iframe src="slider-html" height=40 edit]

<p></p>

Ось робоча логіка після заміни подій миші подібними подіями вказівника:

1. Користувач натискає на `thumb` -- запускає `pointerdown`.
2. Потім переміщує вказівник -- запускається `pointermove`, а наш код переміщує елемент `thumb`.
    - ...Коли вказівник рухається, він може покинути `thumb` слайдера, переміщуватися вище або нижче нього. Великий палець повинен рухатися строго горизонтально, залишаючись на одному рівні з вказівником.

У рішенні на основі подій миші, щоб відстежувати всі рухи вказівника, включно з тим, коли він переміщується вище/нижче `thumb`, ми повинні були призначити обробник події `mousemove` для всього `document`.

Однак це не "найчистіше" рішення. Одна з проблем полягає в тому, що коли користувач переміщує вказівник по документу, він може запускати обробники подій (наприклад, `mouseover`) на деяких інших елементах, викликати абсолютно не пов’язані функції інтерфейсу користувача, а ми цього не хочемо.

Це місце, де `setPointerCapture` вступає в гру.

- Ми можемо викликати `thumb.setPointerCapture(event.pointerId)` в обробнику `pointerdown`,
- Тоді майбутні події вказівника до `pointerup/cancel` будуть перенацілені на `thumb`.
- Коли відбувається `pointerup` (перетягування завершено), прив’язка видаляється автоматично, нам не потрібно перейматись за це.

Тому, навіть якщо користувач переміщує вказівник по всьому документу, обробники подій будуть викликатися на `thumb`. Проте, властивості координат об’єктів події, такі як `clientX/clientY`, залишаться правильними - захоплення впливає лише на `target/currentTarget`.

Ось основний код:

```js
thumb.onpointerdown = function(event) {
  // перенацілити всі події вказівника на повзунок (до події pointerup)
  thumb.setPointerCapture(event.pointerId);

  // почати відстеження переміщень вказівника
  thumb.onpointermove = function(event) {
    // переміщення повзунка: всі події перенаправлені на цей обробник
    let newLeft = event.clientX - slider.getBoundingClientRect().left;
    thumb.style.left = newLeft + 'px';
  };

  // завершити відстеження рухів вказівника при pointerup
  thumb.onpointerup = function(event) {
    thumb.onpointermove = null;
    thumb.onpointerup = null;
    // ...також обробити "drag end", якщо потрібно
  };
};

// примітка: не потрібно викликати thumb.releasePointerCapture,
// це відбувається при pointerup автоматично
```

```online
Повне демо:

[iframe src="slider" height=100 edit]

<p></p>

У демо також є додатковий елемент з обробником `onmouseover`, який показує поточну дату.

Зверніть увагу: перетягуючи повзунок, ви можете навести курсор на цей елемент, і його обробник *не* спрацює.

Таким чином, перетягування тепер без побічних ефектів, завдяки `setPointerCapture`.
```



Зрештою, захоплення вказівника дає нам дві переваги:
1. Код стає чистішим, оскільки нам більше не потрібно додавати/видаляти обробники всього `document`. Прив’язка прибирається автоматично..
2. Якщо в документі є інші обробники подій вказівника, вони не будуть випадково ініційовані вказівником, коли користувач перетягує повзунок.

### Події захоплення вказівника

Тут для повноти слід згадати ще одну річ.

Існують дві події, пов’язані із захопленням вказівника:

- `gotpointercapture` спрацьовує, коли елемент використовує `setPointerCapture` для включення захоплення.
- `lostpointercapture` запускається, коли відбувається звільнення від захоплення: або явно за допомогою виклику `releasePointerCapture`, або автоматично під час `pointerup`/`pointercancel`.

## Підсумки

Події вказівника дозволяють обробляти події миші, дотику та пера одночасно, за допомогою одного фрагмента коду.

Події вказівника розширюють події миші. Ми можемо замінити `mouse` на `pointer` в назвах подій і очікувати, що наш код продовжить працювати для миші з кращою підтримкою інших типів пристроїв.

Для перетягування та складних взаємодій дотиком, які браузер може вирішити перехопити та обробити самостійно – не забудьте скасувати типову дію і встановити `touch-action: none` у CSS для елементів, які ми використовуємо.

Додатковими можливостями подій вказівника є:

- Підтримка мультитач за допомогою `pointerId` та `isPrimary`.
- Специфічні властивості пристрою, такі як `pressure`, `width/height` та інші.
- Захоплення вказівника: ми можемо перенацілювати всі події вказівника на певний елемент до `pointerup`/`pointercancel`.

На цей час події вказівника підтримуються в усіх основних браузерах, тому ми можемо безпечно переходити на них, особливо якщо IE10- та Safari 12- не потрібні. І навіть для цих браузерів існують поліфіли, які дозволяють підтримувати події вказівника.
