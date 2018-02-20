---
title: Komponenty
type: guide
order: 11
---

## Czym są komponenty?

Komponenty są jedną z najpotężniejszych cech Vue. Pozwalają na rozszeżenie podstawowego html i enkapsulacje go do ponownego uzycia. Na wysokim poziomie, komponenty są personalizowanymi elementami, do których kompilator Vue dodaje dynamikę. W niektórych przypadkach moga wystepować jako natywny HTML rozszeżony o atrybut `is`.

Każdy komponent jest jednocześnie instancja Vue, więc przyjmuje taki sam obiekt z opcjami (poza kilkoma specjanymi opcjami) i oferuje takie same uchwyty cyli życia.

## Korzystanie z komponentów

### Rejestracja globalna

W poprzednich sekcjach tworzyliśmy nowe instacje Vue za pomocą kodu:

``` js
new Vue({
  el: '#jakis-element',
  // opcje
})
```

Aby zrejestrować komponent globalnie, użyj: `Vue.component(tagName, options)`.
Przykład:

``` js
Vue.component('moj-komponent', {
  // opcje
})
```

<p class="tip">Zauważ, że Vue nie wymusza stosowania reguł [W3C](https://www.w3.org/TR/custom-elements/#concepts) dla nazw tagów użytkownika (wszytsko małymi literami, muszą zawierać dywiz) ale przestrzeganie tej konwencji jest dora praktyką.</p>

Zarejestrownay komponent może być uzyty w szablonie instancji jako tag użytkownika `<moj-komponent></moj-komponent>`. Upewnij się, że komponent jest Zarejestrownay **przed** utworzeniem głównej instancji Vue. Poniżej przykład:

``` html
<div id="example">
  <moj-komponent></moj-komponent>
</div>
```

``` js
// rejestracja
Vue.component('moj-komponent', {
  template: '<div>Komponent użytkownika!</div>'
})

// tworzenie głównej instacji
new Vue({
  el: '#example'
})
```

Wyrenderuje:

``` html
<div id="example">
  <div>Komponent użytkownika!</div>
</div>
```

{% raw %}
<div id="example" class="demo">
  <moj-komponent></moj-komponent>
</div>
<script>
Vue.component('moj-komponent', {
  template: '<div>Komponent użytkownika!</div>'
})
new Vue({ el: '#example' })
</script>
{% endraw %}

### Rejestracja lokalna

Nie musisz rejestrować każdego komponentu globalnie. Możesz utworzyć komponent dostępny tylko w zasięgu innej instancji/komponentu, rejestrując go opcją `components` w instancji:

``` js
var potomek = {
  template: '<div>Komponent użytkownika!</div>'
}

new Vue({
  // ...
  components: {
    // <moj-komponent> będzie dostępny tylko w szablonie elementu nadrzędnego
    'moj-komponent': Child
  }
})
```

Ta sama enkapsulacja dotyczy innych zarejestrowanych funkcjonalności Vue, takich jak dyrektywy.

### Parsowanie szablonów DOM

Korzystając z DOM jako szablonu (np: używając opcji `el` do osadzenia elementu z istniejącą zawartością), napotkasz pewne ograniczenia wynikające z działania HTMLa, ponieważ Vue może podbrać zawartość szablonu **po** parsowaniu i normalizacji przez przeglądarkę. Dzieje się tak dlatego, że elementy jak `<ul>`, `<ol>`, `<table>` i `<select>` mogą się pojawić jedynie wewątrz innych elementów.

Doprowadzi to do problemów podczas używania niestandardowych komponentów z elementami, które mają takie ograniczenia, na przykład:

``` html
<table>
  <moj-row>...</moj-row>
</table>
```

Komponent `<my-row>` zostanie potraktowany jako nieprawidłowa zawartość, to może generować błędy podczas renderowania. Obejściem jest zastosowanie atrybutu `is`:

``` html
<table>
  <tr is="moj-row"></tr>
</table>
```

**Te ograniczenia nie wystepują jeżeli wykorzystujesz szablony łańcuchowe z wymienionych źródeł:**

- `<script type="text/x-template">`
- szablon osadzony lokalnie jako łańcuch JavaScript
- komponenty `.vue`

W związku z tym należy korzystać z szablonów łancuchowych, zawsze kiedy to jest mozliwe.

### `data` musi być funkcją

Większość opcji zdefiniowana w konstruktorze Vue może być wykorzystana w komponencie, z jednym zastrzeżeniem: `data` musi być funkcją. Jeżeli użyjesz kod:

``` js
Vue.component('moj-component', {
  template: '<span>{{ komunikat }}</span>',
  data: {
    komunikat: 'Cześć!'
  }
})
```

Vue zatrzyma się i wyświetli w konsoli komunikat: `data` must be a function for component instances (dla instancji komponentu `data` musi byc funkcją). Żeby lepiej zrozumieć zasady spróbujmy nieco oszukać:

``` html
<div id="example-2">
  <prosty-licznik></prosty-licznik>
  <prosty-licznik></prosty-licznik>
  <prosty-licznik></prosty-licznik>
</div>
```

``` js
var data = { licznik: 0 }

Vue.component('prosty-licznik', {
  template: '<button v-on:click="licznik += 1">{{ licznik }}</button>',
  // data technicznie jest funkcją, więc Vue
  // nie wyświetli błędu i zwróci odwołanie do obiektu
  // dla każdej instancji
  data: function () {
    return data
  }
})

new Vue({
  el: '#example-2'
})
```

{% raw %}
<div id="example-2" class="demo">
  <prosty-licznik></prosty-licznik>
  <prosty-licznik></prosty-licznik>
  <prosty-licznik></prosty-licznik>
</div>
<script>
var data = { licznik: 0 }
Vue.component('prosty-licznik', {
  template: '<button v-on:click="licznik += 1">{{ licznik }}</button>',
  data: function () {
    return data
  }
})
new Vue({
  el: '#example-2'
})
</script>
{% endraw %}

Jeżeli każda z instancji komponentu ma ten sam obiekt `data`, inkrementacja jednego licznika inkrementuje wszystkie!
Since all three component instances share the same `data` object, incrementing one counter increments them all! Ouch. Naprawmy to, zwracając nowy obiekt `data`:

``` js
data: function () {
  return {
    licznik: 0
  }
}
```

Teraz każdy licznik będzie miał własny stan:

{% raw %}
<div id="example-2-5" class="demo">
  <moj-komponent></moj-komponent>
  <moj-komponent></moj-komponent>
  <moj-komponent></moj-komponent>
</div>
<script>
Vue.component('moj-komponent', {
  template: '<button v-on:click="licznik += 1">{{ licznik }}</button>',
  data: function () {
    return {
      licznik: 0
    }
  }
})
new Vue({
  el: '#example-2-5'
})
</script>
{% endraw %}

### Komponowanie komponentów

Komponenty powinny być używane razem, zwykle w relacji rodzic - dziecko: komponent A może użyć komponentu B w jego własnym szablonie. Nieunikniona jest komunikacja między nimi: rodzic może oczekiwać danych od dziecka, a dziecko może informować rodzica co się z nim dzieje. Bardzo ważne jest żeby zachować ich rozdzielność, jeżeli to możliwe z wyraźnie zdefiniowanym interfejsem. Daje to pewność, że każdy z komponentów może być napisany utrzymywany we względnej izolacji co sprawia, że będzie nimi łatwiej zarządzać i potencjalnie będą łatwiejsze do ponownego użycia.

W Vue relacje komponentów rodzic-dziecko można podsumować jako: **props przekazywane w dół, zdarzenia emitowane w górę**. Rodzic przekazuje dane do dziecka za pomocą **props**, a dziecko wysyła wiadomości do rodzica za pomocą **zdarzeń**. Zobaczmy jak to funkcjonuje.

<p style="text-align: center;">
  <img style="width: 300px;" src="/images/props-events.png" alt="props down, events up">
</p>

## Właściwość props

### Przekazywanie danych przez właściwość props

Każda instancja komponentu ma swój **izolowany zasięg**. To oznacza, że nie możesz (i nie powinieneś) odwoływać się bezpośrednio do danych rodzica w szablonie komponentu potomnego. Dane moga być przekazywane w dół do komponentu potomnego korzystając z **props**.

Props jest atrybutem użytkownika do przekazywania informacji z nadrzędnego komponentu. Komponent musi mieć jawnie zadeklarowane właściwości, których oczekuje. Deklaruje się je za pomocą opcji [`props`](../api/#props):

``` js
Vue.component('dziecko', {
  // deklaracja właściwości
  props: ['komunikat'],
  // tak jak 'data', 'props' może być uzyte wewnątrz szablonu
  // i jest również dostępny w vm jako this.message
  template: '<span>{{ komunikat }}</span>'
})
```

Wtedy możemy przekazać zwykły ciąg znaków w ten sposób:

``` html
<dziecko komunikat="Cześć!"></dziecko>
```

Result:

{% raw %}
<div id="prop-example-1" class="demo">
  <dziecko komunikat="Cześć!"></dziecko>
</div>
<script>
new Vue({
  el: '#prop-example-1',
  components: {
    dziecko: {
      props: ['komunikat'],
      template: '<span>{{ komunikat }}</span>'
    }
  }
})
</script>
{% endraw %}

### camelCase kontra kebab-case

Atrybuty HTML uwzględniają wielkość liter, więc korzystając z szablonów nie łańcuchowych pisownia nazw właściwości musi być konwertowana z camelCase na kebeb-case (rozdzielone dywizem):

``` js
Vue.component('dziecko', {
  // camelCase w JavaScript
  props: ['mojKomunikat'],
  template: '<span>{{ mojKomunikat }}</span>'
})
```

``` html
<!-- kebab-case w HTML -->
<dziecko moj-komunikat="Cześć!"></dziecko>
```

Ponownie: te ograniczenia nie wystepują jeżeli wykorzystujesz szablony łańcuchowe.

### Dynamiczna właściwość props

Podobnie do bindowania normalnego atrybutu do wyrażenia, również możesz użyć `v-bind` do dynamicznego binowania właściwości do danych w rodzicu. Jeżeli dane zostaną zaktualizowane w rodzicu, zostaną równiez przekazane dziecku:

``` html
<div>
  <input v-model="komunikatRodzica">
  <br>
  <dziecko v-bind:moj-komunikat="komunikatRodzica"></dziecko>
</div>
```

Możesz również korzystać z wersji skróconej `v-bind`:

``` html
<dziecko :moj-komunikat="komunikatRodzica"></dziecko>
```

Wynik:

{% raw %}
<div id="demo-2" class="demo">
  <input v-model="komunikatRodzica">
  <br>
  <dziecko v-bind:moj-komunikat="komunikatRodzica"></dziecko>
</div>
<script>
new Vue({
  el: '#demo-2',
  data: {
    komunikatRodzica: 'Komunikat od rodzica'
  },
  components: {
    dziecko: {
      props: ['mojKomunikat'],
      template: '<span>{{mojKomunikat}}</span>'
    }
  }
})
</script>
{% endraw %}

Jeżeli chcesz przekazać wszystkie właściwości w obiekcie jako właściwość props, możesz uzyć `v-bind` bez argumentu (`v-bind` zamiast `v-bind:nazwa-wlasciwosci`). Na przykład obiekt `todo`:

``` js
todo: {
  tekst: 'Learn Vue',
  wykonane: false
}
```

Wtedy:

``` html
<todo-item v-bind="todo"></todo-item>
```

będzie ekwiwalentem:

``` html
<todo-item
  v-bind:tekst="todo.tekst"
  v-bind:is-complete="todo.wykonane"
></todo-item>
```

### Literał kontra zawartość dynamiczna

Częstym błędem początkujących jest wstawianie cyfry w składnię literału:

``` html
<!-- ten prop jest łańcuchem znaków, nie liczbą -->
<comp some-prop="1"></comp>
```

Ponieważ prop jest literałem, jego wartość jest przekazywana jako łańcuch znaków `"1"`. Jeżeli chcesz przekazać wartość liczbową, musisz skorzystać z `v-bind`. Wówczas zapis będzie wyglądać w następujący sposób:

``` html
<!-- teraz przekażesz jest liczbę -->
<comp v-bind:some-prop="1"></comp>
```

### Jednokierunkowy przepływ danych
Wszystkie prop tworzą **jednokierunkowe** powiązanie między własnością dziecka a rodzica: gdy właściwość nadrzędna zostanie zaktualizowana, zostanie przekazana dziecku, ale nie na odwrót. Zapobiega to przypadkowemu mutowaniu stanu komponentów rodzica, co mogłoby utrudnić zrozumienie przepływu danych w aplikacji.

Ponadto za każdym razem, gdy składnik rodzic jest aktualizowany, wszystkie prop w elemencie podrzędnym zostaną odświeżone z najnowszą wartością. Oznacza to, że **nie** należy mutować prop wewnątrz elementu potomnego. Jeśli to zrobisz, Vue ostrzeże cię w konsoli.

Zwykle występują dwie sytuacje, w których kusząca jest mutacja prop:

1. Prop jest używana do przekazania wartości początkowej; komponent potomny chce później użyć go jako wartość danych lokalnych.

2. Prop przekazuje wartość wymagającą obróbki.

Właściwe rozwiązania w tych przypadkach to:

1. Zdefiniuj lokalny prop, który wykorzystuje przekazaną wartość prop jako swoją wartość początkową:

  ``` js
  props: ['initialCounter'],
  data: function () {
    return { counter: this.initialCounter }
  }
  ```

2. Zdefiniuj właściwośc wyliczoną, która podda obróbce przekazane dane:

  ``` js
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

<p class="tip">Zauważ, że obiekty i tablice w JavaScript są przekazywane przez odniesienie, więc jeśli prop jest tablicą lub obiektem, mutacja obiektu lub tablicy wewnątrz dziecka **wpłynie** na stan rodzica.</p>

### Walidacja prop

Możliwe jest, że komponent określa wymagania dla prop, które otrzymuje. Jeśli wymaganie nie zostanie spełnione, Vue wyemituje ostrzeżenia. Jest to szczególnie przydatne podczas tworzenia komponentu, który ma być używany przez innych.

Zamiast definiować prop jako tablicę łańcuchów znaków, możesz użyć obiektu z wymaganiami walidacji:

``` js
Vue.component('example', {
  props: {
    // proste sprawdzenie typu 
    // (`null` oznacza akceptację każdego typu)
    propA: Number,
    // wiele dopuszczalnych typów
    propB: [String, Number],
    // wymagany łańcuch znaków
    propC: {
      type: String,
      required: true
    },
    // wartość liczbowa z wartością domyślną
    propD: {
      type: Number,
      default: 100
    },
    // wartości domyślne obiektu / tablicy 
    // powinny być zwrócone przez funkcję wbudowaną
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // funkcja walidacyjna użytkownika
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

`type` może być jednym z natywnych konstruktorów:

- String
- Number
- Boolean
- Function
- Object
- Array
- Symbol

Ponadto, `typ` może być także niestandardową funkcją konstruktora, a asercja zostanie wykonana przy pomocy sprawdzenia `instanceof`.

Gdy walidacja prop nie powiedzie się, Vue wygeneruje ostrzeżenie w konsoli (jeżeli korzystasz z deweloperskiej wersji biblioteki). Zauważ, że prop są sprawdzane __przed__ rozpoczęciem tworzenia instancji komponentu, więc w funkcjach `default` lub` validator` właściwości instancji takie jak `data`,`computed` lub `methods` nie będą dostępne.

## Atrybuty Non-Prop

Atrybuty non-prop są atrybutami przekazywanymi do komponentu, nie mające zdefiniowanego prop korespondującego z nimi.

Chociaż jawnie zdefiniowane prop są preferowane do przekazywania informacji do komponentu potomnego, autorzy bibliotek komponentów nie zawsze mogą przewidzieć konteksty, w których mogą być używane ich komponenty. Dlatego komponenty mogą akceptować dowolne atrybuty, które są dodawane do elementu głównego komponentu.

Przykładowo wyobraź sobie, że korzystamy z komponentu `bs-date-input`. Jest gotowy komponent, z pluginem Bootstrap wymagający atrybutu `data-3d-date-picker` w `input`. Możemy dodać ten atrybut do instacji komponentu:

``` html
<bs-date-input data-3d-date-picker="true"></bs-date-input>
```

Atrybut `data-3d-date-picker="true"` zostanie automatycznie dodany do głównego elementu `bs-date-input`.

### Zastępowanie / scalanie z istniejącymi atrybutami

Tak by wyglądał szblon dla `bs-date-input`:

``` html
<input type="date" class="form-control">
```

Aby zdefiniować skórkę dla pluginu z wybierakiem daty, możemy dodać klasę:

``` html
<bs-date-input
  data-3d-date-picker="true"
  class="date-picker-theme-dark"
></bs-date-input>
```
W tym przypadku otrzymamy dwie wartości `class`:

- `form-control`, ustawiona przez szablon komponentu,
- `date-picker-theme-dark`, dodana do komponentu przez rodzica.

W przypadku większości atrybutów wartość dodana do komponentu zastąpi wartość ustawioną przez jego szablon. Na przykład przekazanie `type="large"` zastąpi `type ="date"` i prawdopodobnie go nadpisze! Na szczęście atrybuty `class` i `style` są trochę mądrzejsze, więc obie wartości są scalane, tworząc końcową wartość: `form-control date-picker-theme-dark`.

## Zdarzenia użytkownika

Dowiedzieliśmy się, że rodzic może przekazywać dane do dziecka za pomocą props, ale w jaki sposób przekazujemy informacje rodzicowi o zdarzeniach? Tutaj pojawia się system zdarzeń użytkownika Vue.

### Wykorzystanie `v-on` ze zdarzeniami użytkownika

Każda instacja Vue implementuje [intefejs zdarzeń](../api/#Instance-Methods-Events), co oznacza, że każda instacja potrafi:

- Nasłuchiwać zdarzeń, korzystając z `$on(eventName)`
- Emitować zdarzenia, korzystając z `$emit(eventName)`

<p class="tip">Zwróć uwagę, że system zdarzeń Vue różni się od [EventTarget API] przeglądarki (https://developer.mozilla.org/en-US/docs/Web/API/EventTarget). Chociaż działają one podobnie, `$on` i `$emit` __nie są__ skrótami `addEventListener` and `dispatchEvent`.</p>

Ponadto komponent nadrzędny, może nasłuchiwać zdarzeń emitowanych przez dziecko. Wystarczy skorzystać z `v-on` bezpośrednio w szablonie gdzie jest wykorzystany komponent potomny.

<p class="tip">Nie możesz użyć `$on` do nasłuchiwania zdarzeń emitowanych przez dziecko. Musisz użyć `v-on` bezpośrednio w szablonie, jak w przykładzie poniżej.</p>

Przykład:

``` html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```

``` js
Vue.component('button-counter', {
  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

{% raw %}
<div id="counter-event-example" class="demo">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  }
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
{% endraw %}

In this example, it's important to note that the child component is still completely decoupled from what happens outside of it. All it does is report information about its own activity, just in case a parent component might care.

### Binding Native Events to Components

There may be times when you want to listen for a native event on the root element of a component. In these cases, you can use the `.native` modifier for `v-on`. For example:

``` html
<my-component v-on:click.native="doTheThing"></my-component>
```

### `.sync` Modifier

> 2.3.0+

In some cases we may need "two-way binding" for a prop - in fact, in Vue 1.x this is exactly what the `.sync` modifier provided. When a child component mutates a prop that has `.sync`, the value change will be reflected in the parent. This is convenient, however it leads to maintenance issues in the long run because it breaks the one-way data flow assumption: the code that mutates child props are implicitly affecting parent state.

This is why we removed the `.sync` modifier when 2.0 was released. However, we've found that there are indeed cases where it could be useful, especially when shipping reusable components. What we need to change is **making the code in the child that affects parent state more consistent and explicit.**

In 2.3.0+ we re-introduced the `.sync` modifier for props, but this time it is only syntax sugar that automatically expands into an additional `v-on` listener:

The following

``` html
<comp :foo.sync="bar"></comp>
```

is expanded into:

``` html
<comp :foo="bar" @update:foo="val => bar = val"></comp>
```

For the child component to update `foo`'s value, it needs to explicitly emit an event instead of mutating the prop:

``` js
this.$emit('update:foo', newValue)
```

### Form Input Components using Custom Events

Custom events can also be used to create custom inputs that work with `v-model`. Remember:

``` html
<input v-model="something">
```

is syntactic sugar for:

``` html
<input
  v-bind:value="something"
  v-on:input="something = $event.target.value">
```

When used with a component, it instead simplifies to:

``` html
<custom-input
  :value="something"
  @input="value => { something = value }">
</custom-input>
```

So for a component to work with `v-model`, it should (these can be configured in 2.2.0+):

- accept a `value` prop
- emit an `input` event with the new value

Let's see it in action with a simple currency input:

``` html
<currency-input v-model="price"></currency-input>
```

``` js
Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)">\
    </span>\
  ',
  props: ['value'],
  methods: {
    // Instead of updating the value directly, this
    // method is used to format and place constraints
    // on the input's value
    updateValue: function (value) {
      var formattedValue = value
        // Remove whitespace on either side
        .trim()
        // Shorten to 2 decimal places
        .slice(
          0,
          value.indexOf('.') === -1
            ? value.length
            : value.indexOf('.') + 3
        )
      // If the value was not already normalized,
      // manually override it to conform
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      // Emit the number value through the input event
      this.$emit('input', Number(formattedValue))
    }
  }
})
```

{% raw %}
<div id="currency-input-example" class="demo">
  <currency-input v-model="price"></currency-input>
</div>
<script>
Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
  props: ['value'],
  methods: {
    updateValue: function (value) {
      var formattedValue = value
        .trim()
        .slice(
          0,
          value.indexOf('.') === -1
            ? value.length
            : value.indexOf('.') + 3
        )
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      this.$emit('input', Number(formattedValue))
    }
  }
})
new Vue({
  el: '#currency-input-example',
  data: { price: '' }
})
</script>
{% endraw %}

The implementation above is pretty naive though. For example, users are allowed to enter multiple periods and even letters sometimes - yuck! So for those that want to see a non-trivial example, here's a more robust currency filter:

<iframe width="100%" height="300" src="https://jsfiddle.net/chrisvfritz/1oqjojjx/embedded/result,html,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Customizing Component `v-model`

> New in 2.2.0+

By default, `v-model` on a component uses `value` as the prop and `input` as the event, but some input types such as checkboxes and radio buttons may want to use the `value` prop for a different purpose. Using the `model` option can avoid the conflict in such cases:

``` js
Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean,
    // this allows using the `value` prop for a different purpose
    value: String
  },
  // ...
})
```

``` html
<my-checkbox v-model="foo" value="some value"></my-checkbox>
```

The above will be equivalent to:

``` html
<my-checkbox
  :checked="foo"
  @change="val => { foo = val }"
  value="some value">
</my-checkbox>
```

<p class="tip">Note that you still have to declare the `checked` prop explicitly.</p>

### Non Parent-Child Communication

Sometimes two components may need to communicate with one-another but they are not parent/child to each other. In simple scenarios, you can use an empty Vue instance as a central event bus:

``` js
var bus = new Vue()
```
``` js
// in component A's method
bus.$emit('id-selected', 1)
```
``` js
// in component B's created hook
bus.$on('id-selected', function (id) {
  // ...
})
```

In more complex cases, you should consider employing a dedicated [state-management pattern](state-management.html).

## Content Distribution with Slots

When using components, it is often desired to compose them like this:

``` html
<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>
```

There are two things to note here:

1. The `<app>` component does not know what content it will receive. It is decided by the component using `<app>`.

2. The `<app>` component very likely has its own template.

To make the composition work, we need a way to interweave the parent "content" and the component's own template. This is a process called **content distribution** (or "transclusion" if you are familiar with Angular). Vue.js implements a content distribution API that is modeled after the current [Web Components spec draft](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md), using the special `<slot>` element to serve as distribution outlets for the original content.

### Compilation Scope

Before we dig into the API, let's first clarify which scope the contents are compiled in. Imagine a template like this:

``` html
<child-component>
  {{ message }}
</child-component>
```

Should the `message` be bound to the parent's data or the child data? The answer is the parent. A simple rule of thumb for component scope is:

> Everything in the parent template is compiled in parent scope; everything in the child template is compiled in child scope.

A common mistake is trying to bind a directive to a child property/method in the parent template:

``` html
<!-- does NOT work -->
<child-component v-show="someChildProperty"></child-component>
```

Assuming `someChildProperty` is a property on the child component, the example above would not work. The parent's template is not aware of the state of a child component.

If you need to bind child-scope directives on a component root node, you should do so in the child component's own template:

``` js
Vue.component('child-component', {
  // this does work, because we are in the right scope
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      someChildProperty: true
    }
  }
})
```

Similarly, distributed content will be compiled in the parent scope.

### Single Slot

Parent content will be **discarded** unless the child component template contains at least one `<slot>` outlet. When there is only one slot with no attributes, the entire content fragment will be inserted at its position in the DOM, replacing the slot itself.

Anything originally inside the `<slot>` tags is considered **fallback content**. Fallback content is compiled in the child scope and will only be displayed if the hosting element is empty and has no content to be inserted.

Suppose we have a component called `my-component` with the following template:

``` html
<div>
  <h2>I'm the child title</h2>
  <slot>
    This will only be displayed if there is no content
    to be distributed.
  </slot>
</div>
```

And a parent that uses the component:

``` html
<div>
  <h1>I'm the parent title</h1>
  <my-component>
    <p>This is some original content</p>
    <p>This is some more original content</p>
  </my-component>
</div>
```

The rendered result will be:

``` html
<div>
  <h1>I'm the parent title</h1>
  <div>
    <h2>I'm the child title</h2>
    <p>This is some original content</p>
    <p>This is some more original content</p>
  </div>
</div>
```

### Named Slots

`<slot>` elements have a special attribute, `name`, which can be used to further customize how content should be distributed. You can have multiple slots with different names. A named slot will match any element that has a corresponding `slot` attribute in the content fragment.

There can still be one unnamed slot, which is the **default slot** that serves as a catch-all outlet for any unmatched content. If there is no default slot, unmatched content will be discarded.

For example, suppose we have an `app-layout` component with the following template:

``` html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

Parent markup:

``` html
<app-layout>
  <h1 slot="header">Here might be a page title</h1>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <p slot="footer">Here's some contact info</p>
</app-layout>
```

The rendered result will be:

``` html
<div class="container">
  <header>
    <h1>Here might be a page title</h1>
  </header>
  <main>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </main>
  <footer>
    <p>Here's some contact info</p>
  </footer>
</div>
```

The content distribution API is a very useful mechanism when designing components that are meant to be composed together.

### Scoped Slots

> New in 2.1.0+

A scoped slot is a special type of slot that functions as a reusable template (that can be passed data to) instead of already-rendered-elements.

In a child component, pass data into a slot as if you are passing props to a component:

``` html
<div class="child">
  <slot text="hello from child"></slot>
</div>
```

In the parent, a `<template>` element with a special attribute `slot-scope` must exist, indicating that it is a template for a scoped slot. The value of `slot-scope` will be used as the name of a temporary variable that holds the props object passed from the child:

``` html
<div class="parent">
  <child>
    <template slot-scope="props">
      <span>hello from parent</span>
      <span>{{ props.text }}</span>
    </template>
  </child>
</div>
```

If we render the above, the output will be:

``` html
<div class="parent">
  <div class="child">
    <span>hello from parent</span>
    <span>hello from child</span>
  </div>
</div>
```

> In 2.5.0+, `slot-scope` is no longer limited to `<template>` and can be used on any element or component.

A more typical use case for scoped slots would be a list component that allows the component consumer to customize how each item in the list should be rendered:

``` html
<my-awesome-list :items="items">
  <!-- scoped slot can be named too -->
  <li
    slot="item"
    slot-scope="props"
    class="my-fancy-item">
    {{ props.text }}
  </li>
</my-awesome-list>
```

And the template for the list component:

``` html
<ul>
  <slot name="item"
    v-for="item in items"
    :text="item.text">
    <!-- fallback content here -->
  </slot>
</ul>
```

#### Destructuring

`slot-scope`'s value is in fact a valid JavaScript expression that can appear in the argument position of a function signature. This means in supported environments (in single-file components or in modern browsers) you can also use ES2015 destructuring in the expression:

``` html
<child>
  <span slot-scope="{ text }">{{ text }}</span>
</child>
```

## Dynamic Components

You can use the same mount point and dynamically switch between multiple components using the reserved `<component>` element and dynamically bind to its `is` attribute:

``` js
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```

``` html
<component v-bind:is="currentView">
  <!-- component changes when vm.currentView changes! -->
</component>
```

If you prefer, you can also bind directly to component objects:

``` js
var Home = {
  template: '<p>Welcome home!</p>'
}

var vm = new Vue({
  el: '#example',
  data: {
    currentView: Home
  }
})
```

### `keep-alive`

If you want to keep the switched-out components in memory so that you can preserve their state or avoid re-rendering, you can wrap a dynamic component in a `<keep-alive>` element:

``` html
<keep-alive>
  <component :is="currentView">
    <!-- inactive components will be cached! -->
  </component>
</keep-alive>
```

Check out more details on `<keep-alive>` in the [API reference](../api/#keep-alive).

## Misc

### Authoring Reusable Components

When authoring components, it's good to keep in mind whether you intend to reuse it somewhere else later. It's OK for one-off components to be tightly coupled, but reusable components should define a clean public interface and make no assumptions about the context it's used in.

The API for a Vue component comes in three parts - props, events, and slots:

- **Props** allow the external environment to pass data into the component

- **Events** allow the component to trigger side effects in the external environment

- **Slots** allow the external environment to compose the component with extra content.

With the dedicated shorthand syntaxes for `v-bind` and `v-on`, the intents can be clearly and succinctly conveyed in the template:

``` html
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="doThis"
  @event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Hello!</p>
</my-component>
```

### Child Component Refs

Despite the existence of props and events, sometimes you might still need to directly access a child component in JavaScript. To achieve this you have to assign a reference ID to the child component using `ref`. For example:

``` html
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>
```

``` js
var parent = new Vue({ el: '#parent' })
// access child component instance
var child = parent.$refs.profile
```

When `ref` is used together with `v-for`, the ref you get will be an array containing the child components mirroring the data source.

<p class="tip">`$refs` are only populated after the component has been rendered, and it is not reactive. It is only meant as an escape hatch for direct child manipulation - you should avoid using `$refs` in templates or computed properties.</p>

### Async Components

In large applications, we may need to divide the app into smaller chunks and only load a component from the server when it's actually needed. To make that easier, Vue allows you to define your component as a factory function that asynchronously resolves your component definition. Vue will only trigger the factory function when the component actually needs to be rendered and will cache the result for future re-renders. For example:

``` js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // Pass the component definition to the resolve callback
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

The factory function receives a `resolve` callback, which should be called when you have retrieved your component definition from the server. You can also call `reject(reason)` to indicate the load has failed. The `setTimeout` here is for demonstration; how to retrieve the component is up to you. One recommended approach is to use async components together with [Webpack's code-splitting feature](https://webpack.js.org/guides/code-splitting/):

``` js
Vue.component('async-webpack-example', function (resolve) {
  // This special require syntax will instruct Webpack to
  // automatically split your built code into bundles which
  // are loaded over Ajax requests.
  require(['./my-async-component'], resolve)
})
```

You can also return a `Promise` in the factory function, so with Webpack 2 + ES2015 syntax you can do:

``` js
Vue.component(
  'async-webpack-example',
  // The `import` function returns a `Promise`.
  () => import('./my-async-component')
)
```

When using [local registration](components.html#Local-Registration), you can also directly provide a function that returns a `Promise`:

``` js
new Vue({
  // ...
  components: {
    'my-component': () => import('./my-async-component')
  }
})
```

<p class="tip">If you're a <strong>Browserify</strong> user that would like to use async components, its creator has unfortunately [made it clear](https://github.com/substack/node-browserify/issues/58#issuecomment-21978224) that async loading "is not something that Browserify will ever support." Officially, at least. The Browserify community has found [some workarounds](https://github.com/vuejs/vuejs.org/issues/620), which may be helpful for existing and complex applications. For all other scenarios, we recommend using Webpack for built-in, first-class async support.</p>

### Advanced Async Components

> New in 2.3.0+

Starting in 2.3.0+ the async component factory can also return an object of the following format:

``` js
const AsyncComp = () => ({
  // The component to load. Should be a Promise
  component: import('./MyComp.vue'),
  // A component to use while the async component is loading
  loading: LoadingComp,
  // A component to use if the load fails
  error: ErrorComp,
  // Delay before showing the loading component. Default: 200ms.
  delay: 200,
  // The error component will be displayed if a timeout is
  // provided and exceeded. Default: Infinity.
  timeout: 3000
})
```

Note that when used as a route component in `vue-router`, these properties will be ignored because async components are resolved upfront before the route navigation happens. You also need to use `vue-router` 2.4.0+ if you wish to use the above syntax for route components.

### Component Naming Conventions

When registering components (or props), you can use kebab-case, camelCase, or PascalCase.

``` js
// in a component definition
components: {
  // register using kebab-case
  'kebab-cased-component': { /* ... */ },
  // register using camelCase
  'camelCasedComponent': { /* ... */ },
  // register using PascalCase
  'PascalCasedComponent': { /* ... */ }
}
```

Within HTML templates though, you have to use the kebab-case equivalents:

``` html
<!-- always use kebab-case in HTML templates -->
<kebab-cased-component></kebab-cased-component>
<camel-cased-component></camel-cased-component>
<pascal-cased-component></pascal-cased-component>
```

When using _string_ templates however, we're not bound by HTML's case-insensitive restrictions. That means even in the template, you can reference your components using:

- kebab-case
- camelCase or kebab-case if the component has been defined using camelCase
- kebab-case, camelCase or PascalCase if the component has been defined using PascalCase

``` js
components: {
  'kebab-cased-component': { /* ... */ },
  camelCasedComponent: { /* ... */ },
  PascalCasedComponent: { /* ... */ }
}
```

``` html
<kebab-cased-component></kebab-cased-component>

<camel-cased-component></camel-cased-component>
<camelCasedComponent></camelCasedComponent>

<pascal-cased-component></pascal-cased-component>
<pascalCasedComponent></pascalCasedComponent>
<PascalCasedComponent></PascalCasedComponent>
```

This means that the PascalCase is the most universal _declaration convention_ and kebab-case is the most universal _usage convention_.

If your component isn't passed content via `slot` elements, you can even make it self-closing with a `/` after the name:

``` html
<my-component/>
```

Again, this _only_ works within string templates, as self-closing custom elements are not valid HTML and your browser's native parser will not understand them.

### Recursive Components

Components can recursively invoke themselves in their own template. However, they can only do so with the `name` option:

``` js
name: 'unique-name-of-my-component'
```

When you register a component globally using `Vue.component`, the global ID is automatically set as the component's `name` option.

``` js
Vue.component('unique-name-of-my-component', {
  // ...
})
```

If you're not careful, recursive components can also lead to infinite loops:

``` js
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```

A component like the above will result in a "max stack size exceeded" error, so make sure recursive invocation is conditional (i.e. uses a `v-if` that will eventually be `false`).

### Circular References Between Components

Let's say you're building a file directory tree, like in Finder or File Explorer. You might have a `tree-folder` component with this template:

``` html
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>
```

Then a `tree-folder-contents` component with this template:

``` html
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```

When you look closely, you'll see that these components will actually be each other's descendent _and_ ancestor in the render tree - a paradox! When registering components globally with `Vue.component`, this paradox is resolved for you automatically. If that's you, you can stop reading here.

However, if you're requiring/importing components using a __module system__, e.g. via Webpack or Browserify, you'll get an error:

```
Failed to mount component: template or render function not defined.
```

To explain what's happening, let's call our components A and B. The module system sees that it needs A, but first A needs B, but B needs A, but A needs B, etc, etc. It's stuck in a loop, not knowing how to fully resolve either component without first resolving the other. To fix this, we need to give the module system a point at which it can say, "A needs B _eventually_, but there's no need to resolve B first."

In our case, let's make that point the `tree-folder` component. We know the child that creates the paradox is the `tree-folder-contents` component, so we'll wait until the `beforeCreate` lifecycle hook to register it:

``` js
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue')
}
```

Problem solved!

### Inline Templates

When the `inline-template` special attribute is present on a child component, the component will use its inner content as its template, rather than treating it as distributed content. This allows more flexible template-authoring.

``` html
<my-component inline-template>
  <div>
    <p>These are compiled as the component's own template.</p>
    <p>Not parent's transclusion content.</p>
  </div>
</my-component>
```

However, `inline-template` makes the scope of your templates harder to reason about. As a best practice, prefer defining templates inside the component using the `template` option or in a `template` element in a `.vue` file.

### X-Templates

Another way to define templates is inside of a script element with the type `text/x-template`, then referencing the template by an id. For example:

``` html
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>
```

``` js
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

These can be useful for demos with large templates or in extremely small applications, but should otherwise be avoided, because they separate templates from the rest of the component definition.

### Cheap Static Components with `v-once`

Rendering plain HTML elements is very fast in Vue, but sometimes you might have a component that contains **a lot** of static content. In these cases, you can ensure that it's only evaluated once and then cached by adding the `v-once` directive to the root element, like this:

``` js
Vue.component('terms-of-service', {
  template: '\
    <div v-once>\
      <h1>Terms of Service</h1>\
      ... a lot of static content ...\
    </div>\
  '
})
```
