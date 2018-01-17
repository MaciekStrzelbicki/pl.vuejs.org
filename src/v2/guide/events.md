---
title: Zarządzanie zdarzeniami
type: guide
order: 9
---

## Nasłuchiwanie zdarzeń

Możemy uzyć dyrektywy `v-on` do nasłuchiwania zdarzeń w DOM oraz do uruchamiania JavaScriptu w odpowiedzi na nie.

Przykład:

``` html
<div id="example-1">
  <button v-on:click="licznik += 1">Dodaj 1</button>
  <p>Przycisk został wciśnięty {{ licznik }} razy.</p>
</div>
```
``` js
var example1 = new Vue({
  el: '#example-1',
  data: {
    licznik: 0
  }
})
```

Wynik:

{% raw %}
<div id="example-1" class="demo">
  <button v-on:click="licznik += 1">Dodaj 1</button>
  <p>Przycisk został wciśnięty {{ licznik }} razy.</p>
</div>
<script>
var example1 = new Vue({
  el: '#example-1',
  data: {
    licznik: 0
  }
})
</script>
{% endraw %}

## Metody do zarządzania zdarzeniami

Logika wielu metod zarządzania zdarzeniami jest bardziej skomplikowana, więc utrzymanie JavaScript w wartości atrybutu `v-on` nie jest możliwe. Daltego `v-on` przyjmuje równiez nazwę funkcji, którą chcesz wywołać.

Przykład:

``` html
<div id="example-2">
  <!-- `powitanie` jest nazwą funkcji zdefiniowanej poniżej  -->
  <button v-on:click="powitanie">Powitanie</button>
</div>
```

``` js
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // definuj funkcje w obiekcie `methods`
  methods: {
    powitanie: function (event) {
      // `this` wewnątrz obiektu `methods` wskazuje na instancję Vue
      alert('Hello ' + this.name + '!')
      // `event` jest natywnym zdarzeniem w DOM
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})

// możesz również wywołać tą funkcję z JavaScript
example2.powitanie() // => 'Hello Vue.js!'
```

Wynik:

{% raw %}
<div id="example-2" class="demo">
  <!-- `powitanie` jest nazwą funkcji zdefiniowanej poniżej  -->
  <button v-on:click="powitanie">Powitanie</button>
</div>
<script>
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // definuj funkcje w obiekcie `methods`
  methods: {
    powitanie: function (event) {
      // `this` wewnątrz obiektu `methods` wskazuje na instancję Vue
      alert('Hello ' + this.name + '!')
      // `event` jest natywnym zdarzeniem w DOM
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})
</script>
{% endraw %}

## Funkcje w tagach html

Zamiast bindowania nazwy funkcji, możesz wywołać funkcję JavaScript bezpośrednio w tagu html:

``` html
<div id="example-3">
  <button v-on:click="powiedz('Cześć!')">Powiedz: cześć</button>
  <button v-on:click="powiedz('Co?')">Powiedz: co</button>
</div>
```
``` js
new Vue({
  el: '#example-3',
  methods: {
    powiedz: function (komunikat) {
      alert(komunikat)
    }
  }
})
```

Result:
{% raw %}
<div id="example-3" class="demo">
  <button v-on:click="powiedz('Cześć!')">Powiedz: cześć</button>
  <button v-on:click="powiedz('Co?')">Powiedz: co</button>
</div>
<script>
new Vue({
  el: '#example-3',
  methods: {
    powiedz: function (komunikat) {
      alert(komunikat)
    }
  }
})
</script>
{% endraw %}

Czasami zachodzi potrzebny jest dostęp do ogryginalnego zdarzenia w DOM. W tagu html możesz użyć dedykowanej zmiennej `$event`:

``` html
<button v-on:click="warn('Formularz nie został wysłany.', $event)">
  Wyslij
</button>
```

``` js
// ...
methods: {
  warn: function (komunikat, event) {
    // teraz mamy dostęp do natywnego zdarzenia
    if (event) event.preventDefault()
    alert(komunikat)
  }
}
```

## Modyfikatory zdarzeń

Podczas obsługi zdarzenia bardzo często zachodzi potrzeba wywołania `event.preventDefault()` lub `event.stopPropagation()`. Możemy to zrobić łatwo wewnątrz funkcji, ale lepiej jest odizolować logikę od obsługi zdarzeń.

Aby zaadresować ten problem, Vue zapewnia **modyfikatory zdarzeń** dla `v-on`. Wywołanie modyfikatora rozpoczyna się od kropki.

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`

``` html
<!-- propagacja zdarzenia zostanie zatrzymana  -->
<a v-on:click.stop="zrobTo"></a>

<!-- zdarzenie submit nie przeładuje strony -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- modyfikatory moga byc łączone w łańcuchy  -->
<a v-on:click.stop.prevent="zrobTamto"></a>

<!-- sam modyfikator -->
<form v-on:submit.prevent></form>

<!-- użycie trybu capture podczas obsługi zdarzenia -->
<!-- np: zdarzenie jest przechwytywane i przekierowywane do funkcji, gdzie zostanie obsłużone  -->
<div v-on:click.capture="zrobTo">...</div>

<!-- obsługuje zdarzenie jeżeli event.target wskazuje na ten element -->
<!-- np: zdarzenie z elementu potomengo nie zostanie obsłużone -->
<div v-on:click.self="zrobTo">...</div>
```

<p class="tip">Kolejność użycia modyfikatorów jest istotna, ponieważ kod jest generowany w tej samej kolejności. Dlatego użycie `@click.prevent.self` blokuje **wszystkie kliknięcia**, a `@click.self.prevent` blokuje kliknięcia tylko z tego elementu.</p>

> Nowość 2.1.4+

``` html
<!-- obsługa zdarzenia klik zostanie wykonana tylko raz -->
<a v-on:click.once="zrobTo"></a>
```

W przeciwieństwie do innych modyfikatorów, które są przeznaczone wyłącznie dla natywnych zdarzeń DOM, modyfikator `.once` może być użyty również w [komponentach](components.html#Using-v-on-with-Custom-Events). Jeżeli jeszcze nie znasz komponentów, nie przejmuj się tym teraz.

## Modyfikatory zdarzeń klawiatury

Nasłuchując zdarzeń klawiatury, często sprawdza się kod klawisza. Vue obsługuje modyfikatory klawiatury dla dyrektywy `v-on`:

``` html
<!-- wywoła `vm.submit()`, tylko gdy `keyCode` wynosi `13` -->
<input v-on:keyup.13="wyślij">
```

Zapamiętanie wszystkich wartości `keyCode` jest uciążliwe, więc w Vue są zaimplemetowane skróty do najczęsściej wykorzystywanych klawiszy:

``` html
<!-- jak powyżej -->
<input v-on:keyup.enter="wyślij">

<!-- działa również ze skróconą wersją -->
<input @keyup.enter="wyślij">
```

Pełna lista skrótów do najczęsściej wykorzystywanych klawiszy:

- `.enter`
- `.tab`
- `.delete` (działa zarówno na "delete" jak i "backspace")
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

Możesz również [zdefiniować własne skróty do klawiszy](../api/#keyCodes) w globalnym obiekcie `config.keyCodes`:

``` js
// włącza skrót `v-on:keyup.f1`
Vue.config.keyCodes.f1 = 112
```

### Automatyczne modyfikatory klawiatury

> Nowość w 2.5.0+

Możesz również używać nazwy klawisza, pełna listę znajdziesz tu: [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) paniętaj zmianie pisowni na kebab-case:

``` html
<input @keyup.page-down="onPageDown">
```

W powyższym przykładzie, obsługa zdarzenia zostanie wywołana jeżeli `$event.key === 'PageDown'`.

<p class="tip">Niektóre klawisze (`.ecs` i wszystkie kursory) maja niespójne wartości `key` w IE9, wbudowane skróty są wskazane, jeżeli zależy Ci na obsłudzie IE9.</p>

## Systemowe modyfikatory klawiatury

> Nowość 2.1.0+

Możesz uzyć modyfikatorów do uruchomienia zdarzenia myszy lub klawiatury z wciśnietym klawiszem modyfikującym:

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

> Uwaga: Na klawiaturze Macintosha, meta to klawisz command (⌘). W Windows, meta to klawisz windows (⊞). W klawiaturach Sun Microsystems meta jest oznaczona rombem (◆). W niektórych klawiaturach, zwłaszcza w urządzeniach MIT i Lisp oraz ich następcach np: Knight, space-cadet, klawisz meta jest oznaczony “META” lub “Meta”.

Przykład:

```html
<!-- Alt + C -->
<input @keyup.alt.67="wyczysc">

<!-- Ctrl + Click -->
<div @click.ctrl="zrobCos">Zrób coś</div>
```

<p class="tip">Zwróc uwagę, że klawisze modyfikatory działają inaczej niż pozostałe, korzystając ze zdarzenia `keyup`, klawisz modyfikujący musi być wciśnięty aby wywołać zdarzenie. Innymi słowy zdarzenie `keyup.ctrl` wystąpi tylko jeżeli puściśz klawisz `ctrl` mając wciśnięty klawisz `ctrl`. Inaczej zdarzenie nie zostanie wywołane.</p>

### Modyfikator `.exact`

> Nowość w 2.5.0+

Modyfikator `.exact` pozwala na kombinację systemowych modyfikatorów potrzebnych do wywołania zdarzenia.

``` html
<!-- zdarzenie zostanie wywołane również, jeżeli będzie to klawisz Alt lub Shift -->
<button @click.ctrl="onClick">A</button>

<!-- zdarzenie zostanie wywołane tylko, gdy klawisz Ctrl zostanie wciśniety -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- zdarzenie zostanie wywołane tylko po kliknięciu bez systemowych modyfikatorów -->
<button @click.exact="onClick">A</button>
```

### Modyfikatory klawiszy myszy

> Nowość w 2.2.0+

- `.left`
- `.right`
- `.middle`

Te modyfikatory ograniczają procedurę obsługi do zdarzeń wywoływanych przez konkretny przycisk myszy.

## Dlaczego nasłuchiwać zdarzeń w HTML?

Podejście, aby nasłuchiwać zdarzeń w html może wywoływać zdziwienie, ponieważ narusza starą zasadę "rozdzielenia zależności". Bądź spokojny, wszystkie funkcje Vue obsługujące zdarzenia są bezpośrednio bindowane do ViewModel, który obsługuje aktualny widok i nie powoduje to żadnych kłopotów w utrzymaniu. Jest kilka korzyści z używania `v-on`:

1. Przeglądając szablon HTML łatwiej jest znaleźć implemetację funkcji w JS.

2. Ponieważ nie musisz recznie dołączać detektorów zdarzeń w JS, Twój kod ViewModel zawiera czysta logikę i jest pozbawiony DOM, co ułatwia testowanie.

3. Po usunięciu ViewModel, wszystkie detektory zdarzeń są automatycznie usuwane. Nie musisz się martwić o ich ręczne czyszczenie.
