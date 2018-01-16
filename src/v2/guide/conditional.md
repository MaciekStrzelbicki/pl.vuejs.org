---
title: Renderowanie Warunkowe
type: guide
order: 7
---

## `v-if`

W bibliotekach bazujących na szablonach, takich jak na przykład Handlebars, piszemy bloki warunkowe w taki sposób:


``` html
<!-- Szablon Handlebars -->
{{#if ok}}
  <h1>Tak</h1>
{{/if}}
```

Żeby osiągnąć ten sam rezultat w Vue używamy dyrektywy `v-if`:

``` html
<h1 v-if="ok">Tak</h1>
```

Możemy także dodać warunek alternatywny z pomocą `v-else`:

``` html
<h1 v-if="ok">Tak</h1>
<h1 v-else>Nie</h1>
```

### Grupowanie warunków z użyciem `v-if` w tagu `<template>`

Ponieważ `v-if` jest dyrektywą, musi być podpięta do pojedynczego elementu. Ale co jeżeli chcemy zarządzać w ten sposób więcej niż jednym elementem? W takim przypadku możemy użyć `v-if` w tagu `<template>`, który traktowany będzie jak niewidoczne opakowanie. Ostatecznie wyrenderowany HTML nie będzie zawierał elementu `<template>`.

``` html
<template v-if="ok">
  <h1>Tytuł</h1>
  <p>Paragraf 1</p>
  <p>Paragraf 2</p>
</template>
```

### `v-else`

Żeby użyć alternatywnego warunku dla `v-if` możemy użyć dyrektywy `v-else`:

``` html
<div v-if="Math.random() > 0.5">
  Teraz mnie widzisz
</div>
<div v-else>
  a teraz nie!
</div>
```

Element z `v-else` musi się pojawić od razu za elementem z `v-if` - w przeciwnym wypadku Vue nie rozpozna go jako część warunku blokowego.

### `v-else-if`

> Nowość w 2.1.0+

Dyrektywa `v-else-if` jak sugeruje jej nazwa jest dyrektywą pozwalającą dodać nam kolejny warunek w połączeniu z `v-if`. Możemy jej użyć wielokrotnie:

```html
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Nie A/B/C
</div>
```

Podobnie jak `v-else`, `v-else-if` musi pojawiać się od razu za elementem posiadającym `v-if`, lub `v-else-if`.

### Kontrolowanie wielokrotnego użycia elementów z użyciem `key`

Vue stara się renderować elementy najwydajniej jak jest to możliwe, dlatego często używa ich ponownie zamiast renderować je od początku. Takie zachowanie oprócz przyspieszania działania aplikacji w Vue, może nieść za sobą wiele korzyści. Na przykład, jeżeli chcemy umożliwić użytkownikowi przełączanie się między klikoma rodzajami logowania się.

``` html
<template v-if="loginType === 'nazwaUzytkownika'">
  <label>Login</label>
  <input placeholder="Wprowadź swój login">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Wprowadź swój adres email">
</template>
```

Zmiana `loginType` w powyższym kodzie nie spowoduje usunięcia danych wprowadzonych przez użytkownika. Ponieważ oba szablony używają tych samych elementów, `input` nie jest podmieniany - zmianie ulega tylko jego `placeholder`.

Sprawdź to wpisując w pole input jakiś tekst i klikając przycisk.

{% raw %}
<div id="no-key-example" class="demo">
  <div>
    <template v-if="loginType === 'nazwaUzytkownika'">
      <label>Nazwa użytkownika</label>
      <input placeholder="Wpisz nazwę użytkownika">
    </template>
    <template v-else>
      <label>E-mail</label>
      <input placeholder="Wpisz adres e-mail">
    </template>
  </div>
  <button @click="toggleLoginType">Zmień sposób logowania</button>
</div>
<script>
new Vue({
  el: '#no-key-example',
  data: {
    loginType: 'nazwaUzytkownika'
  },
  methods: {
    toggleLoginType: function () {
      return this.loginType = this.loginType === 'nazwaUzytkownika' ? 'email' : 'nazwaUzytkownika'
    }
  }
})
</script>
{% endraw %}

To zachowanie nie zawsze jest pożądane, więc Vue oferuje nam możliwość powiedzenia naszej aplikacji "Hej! te dwa elementy są zupełnie różne, nie używaj ich ponownie". Po prostu dodaj do nich atrybut `key` z unikalną dla każdego elementu wartością.

``` html
<template v-if="loginType === 'nazwaUzytkownika'">
  <label>Login</label>
  <input placeholder="Wprowadź swój login" key="login-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Wprowadź swój adres email" key="email-input">
</template>
```

Now those inputs will be rendered from scratch each time you toggle. See for yourself:
Teraz, powyższe inputy będą renderowane od nowa za każdym razem kiedy naciśniesz przycisk. Sprawdź sam.

{% raw %}
<div id="key-example" class="demo">
  <div>
    <template v-if="loginType === 'nazwaUzytkownika'">
      <label>Nazwa użytkownika</label>
      <input placeholder="Wpisz nazwę użytkownika" key="username-input">
    </template>
    <template v-else>
      <label>E-mail</label>
      <input placeholder="Wpisz adres e-mail" key="email-input">
    </template>
  </div>
  <button @click="toggleLoginType">Zmień nazwe logowania</button>
</div>
<script>
new Vue({
  el: '#key-example',
  data: {
    loginType: 'nazwaUzytkownika'
  },
  methods: {
    toggleLoginType: function () {
      return this.loginType = this.loginType === 'nazwaUzytkownika' ? 'email' : 'nazwaUzytkownika'
    }
  }
})
</script>
{% endraw %}

Zauważ, że tag `<label>` jest w dalszym ciąglu używany ponownie, ponieważ nie ma swojego atrybutu `key`.

## `v-show`

Inną metodą warunkowego wyświetlania elementów jest dyrektywa `v-show`. Sposób jej użycia jest bardzo podobny do wcześniejszej metody:

``` html
<h1 v-show="ok">Cześć!</h1>
```

Różnica polega na tym, że element z `v-show` będzie renderowany zawsze i pozostanie widoczny w kodzie DOM, `v-show` przełącza wartość atrybutu `display` w CSSie.

<p class="tip">Zauważ, że `v-show` nie działa w połączeniu z tagiem `<template>`, nie działa też w połączeniu z `v-else`</p>

## `v-if` kontra `v-show`

`v-if` jest "prawdziwym" warunkowym renderowaniem, ponieważ zapewnia nam, że wszystkie detektory zdarzeń oraz komponenty, które są dziećmi warunkowego bloku są poprawnie usuwane i ponownie tworzone podczas przełączania stanu.  

`v-if` jest również **leniwe**: nie stanie się nic, jeżeli warunek nie jest spełniony przy pierwszym renderowaniu - blok warunkowy zostanie wyrenderowany dopiero kiedy warunek zostanie spełniony.

Porównując te dwa sposoby, `v-show` jest znacznie mniej skomplikowane - element jest renderowany zawsze, niezależnie od początkowego warunku. Jego widoczność jest zmieniana poprzez przełączanie wartości w CSS.

Generalnie `v-if` jest bardziej obciążający przy przełączaniu klas, natomiast `v-show` jest bardziej obciążający przy pierwszym renderowaniu. W związku z tym używaj `v-show` jeżeli potrzebujesz coś czesto przełączać, a `v-if` jeżeli warunek nie powinien się zbyt często zmieniać.

## `v-if` w połączeniu z `v-for`

Korzystając z `v-if` razem z `v-for`, nalezy pamietać, że `v-for` ma wyższy priorytet niż `v-if`. Sprawdź <a href="../guide/list.html#V-for-and-v-if">przewodnik renderowania list</a>, aby poznać więcej szczegółów.
