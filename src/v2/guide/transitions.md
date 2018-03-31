---
title: Efekty wejścia/wyjścia i efekty dla list
type: guide
order: 201
---

## Przegląd

Vue oferuje różne sposoby zastosowania efektów przejścia, gdy elementy są wstawiane, aktualizowane lub usuwane z DOM. Obejmuje to narzędzia do:

- automatycznego zastosowania klasy dla przejść i animacji CSS
- integracja zewnętrznych bibliotek animacji CSS, takich jak Animate.css
- wykorzystana JavaScript, do bezpośredniej manipulacji DOM podczas przechwytywania przejścia
- integracji zewnętrznych bibliotek animacji, takich jak Velocity.js

Na tej stronie omawiamy wyłącznie efekty wejścia, wyjścia oraz efekty dla list, więcej znajdziesz w następnej sekcji [zarządzanie stanem przejścia](transitioning-state.html).

## Przejście dla pojedyńczego elementu/komponentu

Vue udostępnia komponent owijający `transition`, który pozwala dodawać przejścia wejścia/wyjścia dla dowolnego elementu lub komponentu w następujących kontekstach:

- renderowanie warunkowe (używając `v-if`)
- Wyświetlanie warunkowe (za pomocą `v-show`)
- Dynamiczne komponenty
- Węzły główne komponentu

Działa następująco:

``` html
<div id="demo">
  <button v-on:click="show = !show">
    Przełącz
  </button>
  <transition name="fade">
    <p v-if="show">Cześć</p>
  </transition>
</div>
```

``` js
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```

``` css
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to /* .fade-leave-active poniżej wersji 2.1.8 */ {
  opacity: 0
}
```

{% raw %}
<div id="demo">
  <button v-on:click="show = !show">
    Przełącz
  </button>
  <transition name="demo-transition">
    <p v-if="show">Cześć</p>
  </transition>
</div>
<script>
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
</script>
<style>
.demo-transition-enter-active, .demo-transition-leave-active {
  transition: opacity .5s
}
.demo-transition-enter, .demo-transition-leave-to {
  opacity: 0
}
</style>
{% endraw %}

Jeżeli element jest opakowany w komponent `transition`, jest wstawiany lun usuwany, oto jak się to dzieje:

1. Vue automatycznie wykrywa, czy element docelowy ma zastosowane przejścia lub animacje CSS. Jeśli tak, klasy przejściowe CSS zostaną dodane / usunięte w odpowiednim momencie.

2. jeżeli komponent `transition` dostarczy [uchwyty JavaScript](#JavaScript-Hooks), będą wywołane we właściwym momencie

3. Jeżeli przejścia / animacje CSS zostaną wykryte, a uchyty JavaScript nie, operacje wstawienia / usunięcia z DOMu zostaną wywołane w następnej klatce (Zauważ: mowa tu o klatkach animacji w przeglądarce, które się różnią od koncepji Vue `nextTick`).

### Klasy przejścia

Jest sześć klas dodawanych do przejść wejścia / wyjścia.

1. `v-enter`: Początkowy stan dla wejścia. Dodawana przed wstawieniem elementu, usuwana jedną klatkę po wstawieniu elementu.

2. `v-enter-active`: Stan aktywny dla wejścia. Istnieje podczas całej fazy wejścia. Dodawana przed wstawieniem elementu, usuwana po zakończeniu przejścia / animacji. Tej klasy można użyć do zdefiniowania krzywej czasu trwania, opóźnienia i łagodzenia dla efektu weścia.

3. `v-enter-to`: **Dostępna tylko w wersji 2.1.8+.** Stan końcowy dla wejścia. Dodawany klatkę przed wstawieniem elementu (w tym samym momencie `v-emter` jest usuwany), usuwany po zakończeniu animacji / przejścia.

4. `v-leave`: Początkowy stan wyjścia. Dodawana natychmiast po uruchomieniu efektu wyjścia, usuwana po jednej klatce.

5. `v-leave-active`: Stan aktywny dla wyjścia. Dodawana zaraz po wejściu w fazę wyjścia, po wyzwoleniu przejścia, usuwana po zakońćzeniu przejścia / animacji.  Applied during the entire leaving phase. Added immediately when leave transition is triggered, removed when the transition/animation finishes. Tej klasy można użyć do zdefiniowania krzywej czasu trwania, opóźnienia i łagodzenia dla efektu weścia.

6. `v-leave-to`: **Dostępna tylko w wersji 2.1.8+.** Końcowy stan dla wyjścia. Dodawana klatkę po uruchomieniu efektu wyjścia (w tym samym momencie usuwana jest `v-leave`), usuwana po zakończeniu przejścia / animacji.

![Transition Diagram](/images/transition.png)

Każda z tych klas będzie poprzedzona nazwą przejścia. Prefiks `v-` jest wstawiany jako domyślny dla elementu `<transition>` bez nazwy. Jeżeli np. użyjesz `<transition name="moje-przejscie">`, zamiast klasy `v-enter` uzyskasz klasę `moje-przejście-enter`.

`v-enter-active` i `v-leave-active` daje możliwość określenia różnych krzywych dynamiki wejścia / wyjścia, co będzie widoczne w prykładach kolejnej sekcji.

### CSS Transitions


Jeden z najczęstszych typów przejść wykorzystuje przejścia CSS. Oto przykład:

``` html
<div id="example-1">
  <button @click="show = !show">
    Przełącz renderowanie
  </button>
  <transition name="slide-fade">
    <p v-if="show">cześć</p>
  </transition>
</div>
```

``` js
new Vue({
  el: '#example-1',
  data: {
    show: true
  }
})
```

``` css
/* Animacje wejścia i wyjścia mogą mieć  */
/* różne czasy trwania */
.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active poniżej wersji 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}
```

{% raw %}
<div id="example-1" class="demo">
  <button @click="show = !show">
    Przełącz renderowanie
  </button>
  <transition name="slide-fade">
    <p v-if="show">cześć</p>
  </transition>
</div>
<script>
new Vue({
  el: '#example-1',
  data: {
    show: true
  }
})
</script>
<style>
.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to {
  transform: translateX(10px);
  opacity: 0;
}
</style>
{% endraw %}

### Animacje CSS

Animacje CSS są dodawane w ten sam sposób co przejścia CSS, jedyną różnicą jest, że `v-enter` nie jest usuwany natychmiast po wstawieniu elemenu, ale w po zdarzeniu `animationend`.

Poniższy przykład pomija prefixowanie css dla czystości przykładu:

``` html
<div id="example-2">
  <button @click="show = !show">Przełącz widoczność</button>
  <transition name="bounce">
    <p v-if="show">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris facilisis enim libero, at lacinia diam fermentum id. Pellentesque habitant morbi tristique senectus et netus.</p>
  </transition>
</div>
```

``` js
new Vue({
  el: '#example-2',
  data: {
    show: true
  }
})
```

``` css
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```

{% raw %}
<div id="example-2" class="demo">
  <button @click="show = !show">Przełącz widoczność</button>
  <transition name="bounce">
    <p v-show="show">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris facilisis enim libero, at lacinia diam fermentum id. Pellentesque habitant morbi tristique senectus et netus.</p>
  </transition>
</div>

<style>
  .bounce-enter-active {
    -webkit-animation: bounce-in .5s;
    animation: bounce-in .5s;
  }
  .bounce-leave-active {
    -webkit-animation: bounce-in .5s reverse;
    animation: bounce-in .5s reverse;
  }
  @keyframes bounce-in {
    0% {
      -webkit-transform: scale(0);
      transform: scale(0);
    }
    50% {
      -webkit-transform: scale(1.5);
      transform: scale(1.5);
    }
    100% {
      -webkit-transform: scale(1);
      transform: scale(1);
    }
  }
  @-webkit-keyframes bounce-in {
    0% {
      -webkit-transform: scale(0);
      transform: scale(0);
    }
    50% {
      -webkit-transform: scale(1.5);
      transform: scale(1.5);
    }
    100% {
      -webkit-transform: scale(1);
      transform: scale(1);
    }
  }
</style>
<script>
new Vue({
  el: '#example-2',
  data: {
    show: true
  }
})
</script>
{% endraw %}

### NIestandardowe klasy użytkownika

Możesz zdefiniować własne klasy przejścia, podając następujące atrybuty:

- `enter-class`
- `enter-active-class`
- `enter-to-class` (2.1.8+)
- `leave-class`
- `leave-active-class`
- `leave-to-class` (2.1.8+)

Nadpiszą one nazwy klas. Jest to szczególnie przydatne, gdy chcesz połączyć system przejść Vue z biblioteką animacji CSS, taką jak [Animate.css](https://daneden.github.io/animate.css/).

Na przykład:

``` html
<link href="https://cdn.jsdelivr.net/npm/animate.css@3.5.1" rel="stylesheet" type="text/css">

<div id="example-3">
  <button @click="show = !show">
    Przełącz wyświetlanie
  </button>
  <transition
    name="klasy uzytkownika"
    enter-active-class="animowany tadam"
    leave-active-class="animowany odbijwPrawo"
  >
    <p v-if="show">Cześć</p>
  </transition>
</div>
```

``` js
new Vue({
  el: '#example-3',
  data: {
    show: true
  }
})
```

{% raw %}
<link href="https://cdn.jsdelivr.net/npm/animate.css@3.5.1" rel="stylesheet" type="text/css">
<div id="example-3" class="demo">
  <button @click="show = !show">
    Przełącz wyświetlanie
  </button>
  <transition
    name="klasy uzytkownika"
    enter-active-class="animowany tadam"
    leave-active-class="animowany odbijwPrawo"
  >
    <p v-if="show">Cześć</p>
  </transition>
</div>
<script>
new Vue({
  el: '#example-3',
  data: {
    show: true
  }
})
</script>
{% endraw %}

### Używanie przejść i animacji jednocześnie

Vue musi dołączyć detektory zdarzeń, aby wiedzieć, kiedy przejście zostało zakończone. Może to być `transitionend` lub `animationend`, w zależności od zastosowanych reguł CSS. Jeśli używasz tylko jednego lub drugiego, Vue może automatycznie wykryć właściwy typ.

Jednak w niektórych przypadkach możesz chcieć użyć przejścia i animacji na tym samym elemencie, na przykład mieć animację CSS wyzwalaną przez Vue, wraz z efektem przejścia CSS po najechaniu myszą. W takich przypadkach będziesz musiał jawnie zadeklarować typ, którym chcesz się zająć w Vue w atrybucie "type", z wartością `animation` lub` transition`.

### Definicja czasu przejścia

> Nowość 2.2.0+

W większości przypadków Vue może automatycznie dowiedzieć się, kiedy przejście zostało zakończone. Domyślnie Vue czeka na pierwsze zdarzenie `transitionend` lub` animationend` na głównym elemencie przejścia. Jednak nie zawsze może to być pożądane - na przykład możemy mieć sekwencję przejścia z choreografią, w której niektóre zagnieżdżone elementy wewnętrzne mają opóźnione przejście lub dłuższy czas trwania przejścia niż główny element.

W takich przypadkach można określić wyraźny czas trwania przejścia (w milisekundach) prop `duration` w komponencie `<transition>`:

``` html
<transition :duration="1000">...</transition>
```

Możesz również zdefiniować różne czasy dla wejścia i wyjścia:

``` html
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```

### Uchwyty JavaScript

Możesz również zdefiniować uchwyty JavaScript w atrybutach:

``` html
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```

``` js
// ...
methods: {
  // --------
  // WEJŚCIE
  // --------

  beforeEnter: function (el) {
    // ...
  },
  // wywołanie zwrotne done jest opcjonalne gdy
  // budujemy efekt bazując na CSS
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },

  // --------
  // WYJŚCIE
  // --------

  beforeLeave: function (el) {
    // ...
  },
  // wywołanie zwrotne done jest opcjonalne gdy
  // budujemy efekt bazując na CSS
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // leaveCancelled jest dostępne wyłącznie z v-show
  leaveCancelled: function (el) {
    // ...
  }
}
```

Uchwyty mogą być używane w kombiancji z animacjami / przejściamu CSS lub same.

<p class="tip">Podczas korzystania z przejść JavaScript-only, **callback `done` jest wymagany dla uchwytów `enter` i `leave`**. W przeciwnym razie uchwyty będą wywoływane synchronicznie, a przejście zakończy się natychmiast.</p>

<p class="tip">Dobrym pomysłem jest również jawne dodanie `v-bind: css ="false"` dla przejść JavaScript-only, dzięki temu Vue pominie wykrywanie CSS. Zapobiega to również przypadkowemu zakłóceniu przejścia przez reguły CSS.</p>

Zatem zobaczmy przykład przejścia Javascript bazującego na Velocity.js

``` html
<!--
Velocity działa bardzo podobnie do jQuery.animate
i jest świetnym sposobem na przejścia bazujące na JavaScript
-->
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="example-4">
  <button @click="show = !show">
    Przełącz
  </button>
  <transition
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
    v-bind:css="false"
  >
    <p v-if="show">
      Demo
    </p>
  </transition>
</div>
```

``` js
new Vue({
  el: '#example-4',
  data: {
    show: false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      Velocity(el, { opacity: 1, fontSize: '1.4em' }, { duration: 300 })
      Velocity(el, { fontSize: '1em' }, { complete: done })
    },
    leave: function (el, done) {
      Velocity(el, { translateX: '15px', rotateZ: '50deg' }, { duration: 600 })
      Velocity(el, { rotateZ: '100deg' }, { loop: 2 })
      Velocity(el, {
        rotateZ: '45deg',
        translateY: '30px',
        translateX: '30px',
        opacity: 0
      }, { complete: done })
    }
  }
})
```

{% raw %}
<div id="example-4" class="demo">
  <button @click="show = !show">
    Przełącz
  </button>
  <transition
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <p v-if="show">
      Demo
    </p>
  </transition>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>
<script>
new Vue({
  el: '#example-4',
  data: {
    show: false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.transformOrigin = 'left'
    },
    enter: function (el, done) {
      Velocity(el, { opacity: 1, fontSize: '1.4em' }, { duration: 300 })
      Velocity(el, { fontSize: '1em' }, { complete: done })
    },
    leave: function (el, done) {
      Velocity(el, { translateX: '15px', rotateZ: '50deg' }, { duration: 600 })
      Velocity(el, { rotateZ: '100deg' }, { loop: 2 })
      Velocity(el, {
        rotateZ: '45deg',
        translateY: '30px',
        translateX: '30px',
        opacity: 0
      }, { complete: done })
    }
  }
})
</script>
{% endraw %}

## Przejścia na początku renderowania

Aby zastosować przejście do początkowego renderowania węzła, dodaj atrybut `appear`:

``` html
<transition appear>
  <!-- ... -->
</transition>
```

Domyślnie będą używane przejścia dla wejścia i wyjścia. Jeśli chcesz, możesz także określić własne klasy CSS:

``` html
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class" (2.1.8+)
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>
```

oraz własne uchyty JavaScript:

``` html
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
>
  <!-- ... -->
</transition>
```

## Przejścia pomiędzy elementami

[Przejścia pomiędzy komponentami](#Transitioning-Between-Components) omówimy później, możesz również dodawać przejścia pomiędzy surowymi elementami korzystając z `v-if`/`v-else`. Jednym z najczęściej wykorzystywanych przejść jest to pomiędzy kontenerem zawierającym listę, a wiadomością opisującą pustą listę:

``` html
<transition>
  <table v-if="items.length > 0">
    <!-- ... -->
  </table>
  <p v-else>Niestety, nic nie znaleziono</p>
</transition>
```

To działa dobrze, ale o jednym należy pamiętać:

<p class="tip">Podczas przełączania między elementami, które mają **tę samą nazwę znacznika**, musisz powiedzieć Vue, że są one odrębnymi elementami, nadając im unikalne atrybuty . W przeciwnym razie kompilator Vue, w celu zwiększenia wydajności, zastąpi tylko zawartość elementu. Nawet jeśli jest to technicznie niepotrzebne, **uważamy, że dobrą praktyką jest zawsze kluczowanie wielu elementów w komponencie `<transition>`. **</p>

For example:

``` html
<transition>
  <button v-if="isEditing" key="save">
    Zapisz
  </button>
  <button v-else key="edit">
    Edytuj
  </button>
</transition>
```

W takich przypadkach można również użyć atrybutu `key` do przejścia między różnymi stanami tego samego elementu. Zamiast używać `v-if` i `v-else`, powyższy przykład może być przepisany jako:

``` html
<transition>
  <button v-bind:key="isEditing">
    {{ isEditing ? 'Zapisz' : 'Edytuj' }}
  </button>
</transition>
```

W rzeczywistości możliwe jest przejście między dowolną liczbą elementów, przez użycie wielu `v-if`s lub powiązanie pojedynczego elementu z właściwością dynamiczną. Na przykład:

``` html
<transition>
  <button v-if="docState === 'saved'" key="saved">
    Edytuj
  </button>
  <button v-if="docState === 'edited'" key="edited">
    Zapisz
  </button>
  <button v-if="docState === 'editing'" key="editing">
    Anuluj
  </button>
</transition>
```

Które można również napisać jako:

``` html
<transition>
  <button v-bind:key="docState">
    {{ buttonMessage }}
  </button>
</transition>
```

``` js
// ...
computed: {
  buttonMessage: function () {
    switch (this.docState) {
      case 'saved': return 'Edit'
      case 'edited': return 'Save'
      case 'editing': return 'Cancel'
    }
  }
}
```

### Tryby przejścia

Wciąż jest jeden problem. Spróbuj kliknąć poniższy przycisk:

{% raw %}
<div id="no-mode-demo" class="demo">
  <transition name="no-mode-fade">
    <button v-if="on" key="on" @click="on = false">
      on
    </button>
    <button v-else key="off" @click="on = true">
      off
    </button>
  </transition>
</div>
<script>
new Vue({
  el: '#no-mode-demo',
  data: {
    on: false
  }
})
</script>
<style>
.no-mode-fade-enter-active, .no-mode-fade-leave-active {
  transition: opacity .5s
}
.no-mode-fade-enter, .no-mode-fade-leave-active {
  opacity: 0
}
</style>
{% endraw %}

Podczas przechodzenia pomiędzy przyciskiem "on" i "off", oba przyciski są renderowane - jeden z nich znika, podczas gdy drugi się pojawia. Jest to domyślne zachowanie `<transition>` - pojawianie i znikanie odbywa się jednocześnie.

Sprawdza się to znakomicie np. gdy elementy poddawane przejściu są pozycjonowane absolutnie jeden na drugim:

{% raw %}
<div id="no-mode-absolute-demo" class="demo">
  <div class="no-mode-absolute-demo-wrapper">
    <transition name="no-mode-absolute-fade">
      <button v-if="on" key="on" @click="on = false">
        on
      </button>
      <button v-else key="off" @click="on = true">
        off
      </button>
    </transition>
  </div>
</div>
<script>
new Vue({
  el: '#no-mode-absolute-demo',
  data: {
    on: false
  }
})
</script>
<style>
.no-mode-absolute-demo-wrapper {
  position: relative;
  height: 18px;
}
.no-mode-absolute-demo-wrapper button {
  position: absolute;
}
.no-mode-absolute-fade-enter-active, .no-mode-absolute-fade-leave-active {
  transition: opacity .5s;
}
.no-mode-absolute-fade-enter, .no-mode-absolute-fade-leave-active {
  opacity: 0;
}
</style>
{% endraw %}

Można do tego dodać efekt przejścia między slajdami:

{% raw %}
<div id="no-mode-translate-demo" class="demo">
  <div class="no-mode-translate-demo-wrapper">
    <transition name="no-mode-translate-fade">
      <button v-if="on" key="on" @click="on = false">
        on
      </button>
      <button v-else key="off" @click="on = true">
        off
      </button>
    </transition>
  </div>
</div>
<script>
new Vue({
  el: '#no-mode-translate-demo',
  data: {
    on: false
  }
})
</script>
<style>
.no-mode-translate-demo-wrapper {
  position: relative;
  height: 18px;
}
.no-mode-translate-demo-wrapper button {
  position: absolute;
}
.no-mode-translate-fade-enter-active, .no-mode-translate-fade-leave-active {
  transition: all 1s;
}
.no-mode-translate-fade-enter, .no-mode-translate-fade-leave-active {
  opacity: 0;
}
.no-mode-translate-fade-enter {
  transform: translateX(31px);
}
.no-mode-translate-fade-leave-active {
  transform: translateX(-31px);
}
</style>
{% endraw %}

Jednoczesne wejście i wyjście nie zawsze jest pożądane, więc Vue oferuje kilka alternatywnych **trybów przejścia**:

- `in-out`: Przejscie jest najpierw uruchamiane na nowym elemencie, a następnie po zakończeniu, bieżący element wychodzi.

- `out-in`: Bieżący element wychodzi najpierw, a po zakończeniu nowy element wchodzi.

Zaktualizujmy przejście dla naszych guzików on/off, dodajmy `out-in`:

``` html
<transition name="fade" mode="out-in">
  <!-- ... buttony ... -->
</transition>
```

{% raw %}
<div id="with-mode-demo" class="demo">
  <transition name="with-mode-fade" mode="out-in">
    <button v-if="on" key="on" @click="on = false">
      on
    </button>
    <button v-else key="off" @click="on = true">
      off
    </button>
  </transition>
</div>
<script>
new Vue({
  el: '#with-mode-demo',
  data: {
    on: false
  }
})
</script>
<style>
.with-mode-fade-enter-active, .with-mode-fade-leave-active {
  transition: opacity .5s
}
.with-mode-fade-enter, .with-mode-fade-leave-active {
  opacity: 0
}
</style>
{% endraw %}

Wystarczyło dodać jeden atrybut, aby naprawić oryginalne przejście bez dodatkowej stylizacji css.

Tryb `in-out` nie jest używany zbyt często, ale czasami może być przydatny dla nieco innego efektu przejścia. Spróbujmy połączyć go z przejściem slide-fade, nad którym pracowaliśmy wcześniej:

{% raw %}
<div id="in-out-translate-demo" class="demo">
  <div class="in-out-translate-demo-wrapper">
    <transition name="in-out-translate-fade" mode="in-out">
      <button v-if="on" key="on" @click="on = false">
        on
      </button>
      <button v-else key="off" @click="on = true">
        off
      </button>
    </transition>
  </div>
</div>
<script>
new Vue({
  el: '#in-out-translate-demo',
  data: {
    on: false
  }
})
</script>
<style>
.in-out-translate-demo-wrapper {
  position: relative;
  height: 18px;
}
.in-out-translate-demo-wrapper button {
  position: absolute;
}
.in-out-translate-fade-enter-active, .in-out-translate-fade-leave-active {
  transition: all .5s;
}
.in-out-translate-fade-enter, .in-out-translate-fade-leave-active {
  opacity: 0;
}
.in-out-translate-fade-enter {
  transform: translateX(31px);
}
.in-out-translate-fade-leave-active {
  transform: translateX(-31px);
}
</style>
{% endraw %}

Fajnie, prawda?

## Przejścia pomiędzy komponentami

Przejście pomiędzy komponentami jest jeszcze prostsze, nie potrzebujemy atrybutu `key`, wystarczy opakować kommponent w [komponent dynamiczny](components.html#Dynamic-Components):

``` html
<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>
```

``` js
new Vue({
  el: '#transition-components-demo',
  data: {
    view: 'v-a'
  },
  components: {
    'v-a': {
      template: '<div>Komponent A</div>'
    },
    'v-b': {
      template: '<div>Komponent B</div>'
    }
  }
})
```

``` css
.component-fade-enter-active, .component-fade-leave-active {
  transition: opacity .3s ease;
}
.component-fade-enter, .component-fade-leave-to
/* .component-fade-leave-active poniżej wersji 2.1.8 */ {
  opacity: 0;
}
```

{% raw %}
<div id="transition-components-demo" class="demo">
  <input v-model="view" type="radio" value="v-a" id="a" name="view"><label for="a">A</label>
  <input v-model="view" type="radio" value="v-b" id="b" name="view"><label for="b">B</label>
  <transition name="component-fade" mode="out-in">
    <component v-bind:is="view"></component>
  </transition>
</div>
<style>
.component-fade-enter-active, .component-fade-leave-active {
  transition: opacity .3s ease;
}
.component-fade-enter, .component-fade-leave-to {
  opacity: 0;
}
</style>
<script>
new Vue({
  el: '#transition-components-demo',
  data: {
    view: 'v-a'
  },
  components: {
    'v-a': {
      template: '<div>Komponent A</div>'
    },
    'v-b': {
      template: '<div>Komponent B</div>'
    }
  }
})
</script>
{% endraw %}

## Przejścia dla list

Jak dodąd dodawaliśmy przejścia do:

- Pojedyńczych węzłów
- Wielu węzłów gdzie tylko jeden był renderowny w danym momencie

A co zrobić, gdy mamy całą listę elementów, które chcemy renderować jednocześnie, na przykład za pomocą `v-for`? W tym przypadku użyjemy komponentu `<transition-group>`. Zanim jednak przyjrzymy się przykładowi, jest kilka rzeczy, o których warto wiedzieć:

- W przeciwieństwie do `<transition>`, domyślnie renderuje on element: `<span>`. Możesz zmienić element, który jest renderowany za pomocą atrybutu `tag`.

- Elementy wewnątrz **zawsze wymagają** unikalnego atrybutu `key`

### Przejścia wejścia/wyjścia dla elementówlisty

Teraz zagłębimy się w przykład, przejścia wejścia i wyjścia, używają tych samych klas CSS, z których korzystaliśmy wcześniej:

``` html
<div id="list-demo">
  <button v-on:click="add">Dodaj</button>
  <button v-on:click="remove">Usuń</button>
  <transition-group name="list" tag="p">
    <span v-for="item in items" v-bind:key="item" class="list-item">
      {{ item }}
    </span>
  </transition-group>
</div>
```

``` js
new Vue({
  el: '#list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
  }
})
```

``` css
.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active, .list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to /* .list-leave-active poniżej wersji 2.1.8 */ {
  opacity: 0;
  transform: translateY(30px);
}
```

{% raw %}
<div id="list-demo" class="demo">
  <button v-on:click="add">Dodaj</button>
  <button v-on:click="remove">Usuń</button>
  <transition-group name="list" tag="p">
    <span v-for="item in items" :key="item" class="list-item">
      {{ item }}
    </span>
  </transition-group>
</div>
<script>
new Vue({
  el: '#list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
  }
})
</script>
<style>
.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active, .list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
</style>
{% endraw %}

W powyższym przykładzie jest jeden błąd, przy dodawaniu i usuwaniu elementu, otaczające go elementy skokowo zmieniają swoją pozycję. Naprawimy to później.

### Animowana zmiana pozycji elementu listy

Komponent `<transition-group>` ma jeszcze jedną sztuczkę w rękawie. Może animować nie tylko wchodzenie i wychodzenie, ale także zmianę pozycji. Jedyną nową koncepcją, którą musisz znać, aby użyć tej funkcji, jest dodanie **klasy `v-move`**, która jest dodawana, gdy elementy zmieniają pozycje. Podobnie jak inne klasy, jego prefiks będzie pasował do wartości podanego atrybutu `name`, możesz również ręcznie określić klasę za pomocą atrybutu` move-class`.

Ta klasa jest najbardziej przydatna do określenia czasu i krzywej przejściowej, jak w poniższym przykładzie:

``` html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="flip-list-demo" class="demo">
  <button v-on:click="shuffle">Przetasuj</button>
  <transition-group name="flip-list" tag="ul">
    <li v-for="item in items" v-bind:key="item">
      {{ item }}
    </li>
  </transition-group>
</div>
```

``` js
new Vue({
  el: '#flip-list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9]
  },
  methods: {
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```

``` css
.flip-list-move {
  transition: transform 1s;
}
```

{% raw %}
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>
<div id="flip-list-demo" class="demo">
  <button v-on:click="shuffle">Przetasuj</button>
  <transition-group name="flip-list" tag="ul">
    <li v-for="item in items" :key="item">
      {{ item }}
    </li>
  </transition-group>
</div>
<script>
new Vue({
  el: '#flip-list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9]
  },
  methods: {
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
</script>
<style>
.flip-list-move {
  transition: transform 1s;
}
</style>
{% endraw %}

Może się to wydawać magiczne, ale pod maską Vue używa techniki animacji o nazwie [FLIP] (https://aerotwist.com/blog/flip-your-animations/), aby płynnie przenosić elementy ze starej pozycji do nowej pozycja za pomocą przejścia.

Możemy połączyć ta technikę z wcześniejsza implementacją, uzyskamy animację przy kazdej zmianie listy!

``` html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="list-complete-demo" class="demo">
  <button v-on:click="shuffle">Przetasuj</button>
  <button v-on:click="add">Dodaj</button>
  <button v-on:click="remove">Usuń</button>
  <transition-group name="list-complete" tag="p">
    <span
      v-for="item in items"
      v-bind:key="item"
      class="list-complete-item"
    >
      {{ item }}
    </span>
  </transition-group>
</div>
```

``` js
new Vue({
  el: '#list-complete-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```

``` css
.list-complete-item {
  transition: all 1s;
  display: inline-block;
  margin-right: 10px;
}
.list-complete-enter, .list-complete-leave-to
/* .list-complete-leave-active poniżej wersji 2.1.8 */ {
  opacity: 0;
  transform: translateY(30px);
}
.list-complete-leave-active {
  position: absolute;
}
```

{% raw %}
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>
<div id="list-complete-demo" class="demo">
  <button v-on:click="shuffle">Przetasuj</button>
  <button v-on:click="add">Dodaj</button>
  <button v-on:click="remove">Usuń</button>
  <transition-group name="list-complete" tag="p">
    <span v-for="item in items" :key="item" class="list-complete-item">
      {{ item }}
    </span>
  </transition-group>
</div>
<script>
new Vue({
  el: '#list-complete-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
</script>
<style>
.list-complete-item {
  transition: all 1s;
  display: inline-block;
  margin-right: 10px;
}
.list-complete-enter, .list-complete-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
.list-complete-leave-active {
  position: absolute;
}
</style>
{% endraw %}

<p class = "tip">Ważną informacją jest to, że przejścia FLIP nie działają z elementami ustawionymi na `display: inline`. Alternatywnie możesz użyć `display: inline-block` lub umieścić elementy w kontekście flex. </ P>

Animacje FLIP nie są ograniczone do jednej osi. Elementy animowane mogą znajdować się w siatce [transitioned too](https://jsfiddle.net/chrisvfritz/sLrhk1bc/):


{% raw %}
<div id="sudoku-demo" class="demo">
  <strong>Leniwe sudoku</strong>
  <p>Wciskaj przycisk losowania, dopóki nie wygrasz.</p>
  <button @click="shuffle">
    Losuj
  </button>
  <transition-group name="cell" tag="div" class="sudoku-container">
    <div v-for="cell in cells" :key="cell.id" class="cell">
      {{ cell.number }}
    </div>
  </transition-group>
</div>
<script>
new Vue({
  el: '#sudoku-demo',
  data: {
    cells: Array.apply(null, { length: 81 })
      .map(function (_, index) {
        return {
          id: index,
          number: index % 9 + 1
        }
      })
  },
  methods: {
    shuffle: function () {
      this.cells = _.shuffle(this.cells)
    }
  }
})
</script>
<style>
.sudoku-container {
  display: flex;
  flex-wrap: wrap;
  width: 238px;
  margin-top: 10px;
}
.cell {
  display: flex;
  justify-content: space-around;
  align-items: center;
  width: 25px;
  height: 25px;
  border: 1px solid #aaa;
  margin-right: -1px;
  margin-bottom: -1px;
}
.cell:nth-child(3n) {
  margin-right: 0;
}
.cell:nth-child(27n) {
  margin-bottom: 0;
}
.cell-move {
  transition: transform 1s;
}
</style>
{% endraw %}

### Przejścia w listach

By communicating with JavaScript transitions through data attributes, it's also possible to stagger transitions in a list:
Komunikując się z przejściami JavaScript za pośrednictwem atrybutów danych, można także wyzwalać przejścia na liście:
``` html
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="staggered-list-demo">
  <input v-model="query">
  <transition-group
    name="staggered-fade"
    tag="ul"
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <li
      v-for="(item, index) in computedList"
      v-bind:key="item.msg"
      v-bind:data-index="index"
    >{{ item.msg }}</li>
  </transition-group>
</div>
```

``` js
new Vue({
  el: '#staggered-list-demo',
  data: {
    query: '',
    list: [
      { msg: 'Bruce Lee' },
      { msg: 'Jackie Chan' },
      { msg: 'Chuck Norris' },
      { msg: 'Jet Li' },
      { msg: 'Kung Fury' }
    ]
  },
  computed: {
    computedList: function () {
      var vm = this
      return this.list.filter(function (item) {
        return item.msg.toLowerCase().indexOf(vm.query.toLowerCase()) !== -1
      })
    }
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    enter: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 1, height: '1.6em' },
          { complete: done }
        )
      }, delay)
    },
    leave: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 0, height: 0 },
          { complete: done }
        )
      }, delay)
    }
  }
})
```

{% raw %}
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>
<div id="example-5" class="demo">
  <input v-model="query">
  <transition-group
    name="staggered-fade"
    tag="ul"
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <li
      v-for="(item, index) in computedList"
      v-bind:key="item.msg"
      v-bind:data-index="index"
    >{{ item.msg }}</li>
  </transition-group>
</div>
<script>
new Vue({
  el: '#example-5',
  data: {
    query: '',
    list: [
      { msg: 'Bruce Lee' },
      { msg: 'Jackie Chan' },
      { msg: 'Chuck Norris' },
      { msg: 'Jet Li' },
      { msg: 'Kung Fury' }
    ]
  },
  computed: {
    computedList: function () {
      var vm = this
      return this.list.filter(function (item) {
        return item.msg.toLowerCase().indexOf(vm.query.toLowerCase()) !== -1
      })
    }
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    enter: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 1, height: '1.6em' },
          { complete: done }
        )
      }, delay)
    },
    leave: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 0, height: 0 },
          { complete: done }
        )
      }, delay)
    }
  }
})
</script>
{% endraw %}

## Przejścia wielokrotnego użytku

Przejścia można ponownie wykorzystać za pomocą systemu komponentów Vue. Aby utworzyć przejście wielokrotnego użytku, wystarczy tylko umieścić komponent `<transition>` lub `<transition-group>` w katalogu głównym, a następnie przekazać dowolne elementy potomne do komponentu przejścia.

Poniżej przykład wykorzystania szablonu komponentu:

``` js
Vue.component('my-special-transition', {
  template: '\
    <transition\
      name="very-special-transition"\
      mode="out-in"\
      v-on:before-enter="beforeEnter"\
      v-on:after-enter="afterEnter"\
    >\
      <slot></slot>\
    </transition>\
  ',
  methods: {
    beforeEnter: function (el) {
      // ...
    },
    afterEnter: function (el) {
      // ...
    }
  }
})
```
A komponenty funkcjonalne wyjątkowo dobrze nadają się do tego zadania:

``` js
Vue.component('my-special-transition', {
  functional: true,
  render: function (createElement, context) {
    var data = {
      props: {
        name: 'very-special-transition',
        mode: 'out-in'
      },
      on: {
        beforeEnter: function (el) {
          // ...
        },
        afterEnter: function (el) {
          // ...
        }
      }
    }
    return createElement('transition', data, context.children)
  }
})
```

## Przejścia sterowane dynamicznie

Tak, nawet przejścia w Vue są sterowane danymi! Najbardziej podstawowy przykład przejścia dynamicznego wiąże atrybut `name` z właściwością dynamiczną.

```html
<transition v-bind:name="transitionName">
  <!-- ... -->
</transition>
```

Może to być przydatne, gdy zdefiniujesz przejścia / animacje CSS, używając konwencji klas przejść z Vue i chcesz przełączać się między nimi.

``` html
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="dynamic-fade-demo" class="demo">
  Fade In: <input type="range" v-model="fadeInDuration" min="0" v-bind:max="maxFadeDuration">
  Fade Out: <input type="range" v-model="fadeOutDuration" min="0" v-bind:max="maxFadeDuration">
  <transition
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <p v-if="show">Witaj!</p>
  </transition>
  <button
    v-if="stop"
    v-on:click="stop = false; show = false"
  >Uruchom animację</button>
  <button
    v-else
    v-on:click="stop = true"
  >Zatrzymaj</button>
</div>
```

``` js
new Vue({
  el: '#dynamic-fade-demo',
  data: {
    show: true,
    fadeInDuration: 1000,
    fadeOutDuration: 1000,
    maxFadeDuration: 1500,
    stop: true
  },
  mounted: function () {
    this.show = false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 1 },
        {
          duration: this.fadeInDuration,
          complete: function () {
            done()
            if (!vm.stop) vm.show = false
          }
        }
      )
    },
    leave: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 0 },
        {
          duration: this.fadeOutDuration,
          complete: function () {
            done()
            vm.show = true
          }
        }
      )
    }
  }
})
```

{% raw %}
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>
<div id="dynamic-fade-demo" class="demo">
  Fade In: <input type="range" v-model="fadeInDuration" min="0" v-bind:max="maxFadeDuration">
  Fade Out: <input type="range" v-model="fadeOutDuration" min="0" v-bind:max="maxFadeDuration">
  <transition
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <p v-if="show">Witaj!</p>
  </transition>
  <button
    v-if="stop"
    v-on:click="stop = false; show = false"
  >Uruchom animację</button>
  <button
    v-else
    v-on:click="stop = true"
  >Zatrzymaj</button>
</div>
<script>
new Vue({
  el: '#dynamic-fade-demo',
  data: {
    show: true,
    fadeInDuration: 1000,
    fadeOutDuration: 1000,
    maxFadeDuration: 1500,
    stop: true
  },
  mounted: function () {
    this.show = false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 1 },
        {
          duration: this.fadeInDuration,
          complete: function () {
            done()
            if (!vm.stop) vm.show = false
          }
        }
      )
    },
    leave: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 0 },
        {
          duration: this.fadeOutDuration,
          complete: function () {
            done()
            vm.show = true
          }
        }
      )
    }
  }
})
</script>
{% endraw %}

Ostatecznym sposobem tworzenia dynamicznych przejść są komponenty, które wykorzystują props do zmiany charakteru przejść. To może brzmieć źle, ale jedynym ograniczeniem jest naprawdę Twoja wyobraźnia.
