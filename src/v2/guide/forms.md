---
title: Bindowanie inputów w formularzach
type: guide
order: 10
---

## Podstawy

Możesz uzyć dyrektywy `v-model` aby utworzyć dwokierunkowe bindowanie danych polach formularza (input, textarea). Poprawny sposób aktualizacji jest wybierany automatycznie na podstawie typu inputa. Chociaż nieco magiczny, "v-model" jest w istocie cukrem składniowym do aktualizacji danych wprowadzanych przez użytkownika, a także specjalnej troski o niektóre przypadki skrajne.

<p class="tip">`v-model` ignoruje wartość początkową atrybutu `value`, `checked` i `selected`. Zawsze traktuje dane instancji Vue jako źródło prawdy. Powinieneś zadeklarować wartość początkową po stronie JavaScript, w opcji `data` twojego komponentu.</p>

<p class="tip" id="vmodel-ime-tip">Dla języków wymagających [IME](https://en.wikipedia.org/wiki/Input_method) (mandaryński, japoński, koreański itp.), zauważ, że `v-model` nie jest aktualizowany podczas kompilacji IME. Jeśli chcesz również uwzględnić te aktualizacje, użyj zdarzenia `input`.</p>

### Tekst

``` html
<input v-model="komunikat" placeholder="wpisz coś">
<p>Komunikat: {{ komunikat }}</p>
```

{% raw %}
<div id="example-1" class="demo">
  <input v-model="komunikat" placeholder="wpisz coś">
  <p>Komunikat: {{ komunikat }}</p>
</div>
<script>
new Vue({
  el: '#example-1',
  data: {
    komunikat: ''
  }
})
</script>
{% endraw %}

### Wielowierszowy tekst

``` html
<span>Wielowierszowy komunikat</span>
<p style="white-space: pre-line;">{{ komunikat }}</p>
<br>
<textarea v-model="komunikat" placeholder="Wpisz coś w kilku wierszach"></textarea>
```

{% raw %}
<div id="example-textarea" class="demo">
  <span>Komunikat w kilku wierszach:</span>
  <p style="white-space: pre-line;">{{ message }}</p>
  <br>
  <textarea v-model="message" placeholder="Wpisz coś w kilku wierszach"></textarea>
</div>
<script>
new Vue({
  el: '#example-textarea',
  data: {
    message: ''
  }
})
</script>
{% endraw %}

{% raw %}
<p class="tip">Interpolacja w polach tekstowych  (<code>&lt;textarea&gt;{{text}}&lt;/textarea&gt;</code>) nie działa, użyj <code>v-model</code>.</p>
{% endraw %}

### Checkbox

Pojedyńczy checkbox, wartość logiczna:

``` html
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ zaznaczony }}</label>
```
{% raw %}
<div id="example-2" class="demo">
  <input type="checkbox" id="checkbox" v-model="zaznaczony">
  <label for="checkbox">{{ zaznaczony }}</label>
</div>
<script>
new Vue({
  el: '#example-2',
  data: {
    zaznaczony: false
  }
})
</script>
{% endraw %}

Kilka checkboxów bindowanych do jednej tablicy:

``` html
<div id='example-3'>
  <input type="checkbox" id="jacek" value="Jacek" v-model="wybraneImiona">
  <label for="jacek">Jacek</label>
  <input type="checkbox" id="marek" value="Marek" v-model="wybraneImiona">
  <label for="marek">Marek</label>
  <input type="checkbox" id="adam" value="Adam" v-model="wybraneImiona">
  <label for="adam">Adam</label>
  <br>
  <span>Checked names: {{ wybraneImiona }}</span>
</div>
```

``` js
new Vue({
  el: '#example-3',
  data: {
    wybraneImiona: []
  }
})
```

{% raw %}
<div id="example-3" class="demo">
  <input type="checkbox" id="jacek" value="Jacek" v-model="wybraneImiona">
  <label for="jacek">Jacek</label>
  <input type="checkbox" id="marek" value="Marek" v-model="wybraneImiona">
  <label for="john">Marek</label>
  <input type="checkbox" id="adam" value="Adam" v-model="wybraneImiona">
  <label for="mike">Adam</label>
  <br>
  <span>Wybrane imiona: {{ wybraneImiona }}</span>
</div>
<script>
new Vue({
  el: '#example-3',
  data: {
    wybraneImiona: []
  }
})
</script>
{% endraw %}

### Radio butony

``` html
<input type="radio" id="one" value="One" v-model="picked">
<label for="one">Pierwszy</label>
<br>
<input type="radio" id="two" value="Two" v-model="picked">
<label for="two">Drugi</label>
<br>
<span>wybrany: {{ wybrany }}</span>
```
{% raw %}
<div id="example-4" class="demo">
  <input type="radio" id="one" value="Pierwszy" v-model="wybrany">
  <label for="one">Pierwszy</label>
  <br>
  <input type="radio" id="two" value="Drugi" v-model="wybrany">
  <label for="two">Drugi</label>
  <br>
  <span>wybrany: {{ wybrany }}</span>
</div>
<script>
new Vue({
  el: '#example-4',
  data: {
    wybrany: ''
  }
})
</script>
{% endraw %}

### Select

Jednokrotny wybór:

``` html
<select v-model="selected">
  <option disabled value="">Wybierz jeden</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<span>Wybrany: {{ wybrany }}</span>
```
``` js
new Vue({
  el: '...',
  data: {
    Wybrany: ''
  }
})
```
{% raw %}
<div id="example-5" class="demo">
  <select v-model="wybrany">
    <option disabled value="">Wybierz jeden</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Wybrany: {{ wybrany }}</span>
</div>
<script>
new Vue({
  el: '#example-5',
  data: {
    wybrany: ''
  }
})
</script>
{% endraw %}

<p class="tip">Jeśli początkowa wartość twojego wyrażenia `v-model` nie pasuje do żadnej z opcji, element `<select>` wyrenderuje się w stanie "niezaznaczonym". W systemie iOS użytkownik nie będzie mógł wybrać pierwszego elementu, ponieważ iOS nie uruchamia w tym przypadku zdarzenia zmiany. Dlatego zaleca się, aby podać opcję wyłączoną z pustą wartością, jak pokazano w powyższym przykładzie.</p>

Wielokrotny wybór (bindowany z tablicą):

``` html
<select v-model="wybrane" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<br>
<span>Wybrane: {{ wybrane }}</span>
```
{% raw %}
<div id="example-6" class="demo">
  <select v-model="wybrane" multiple style="width: 50px;">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
  <span>Wybrane: {{ wybrane }}</span>
</div>
<script>
new Vue({
  el: '#example-6',
  data: {
    wybrane: []
  }
})
</script>
{% endraw %}

Wykorzystanie `v-for` do dynamicznego renderowania:

``` html
<select v-model="wybrane">
  <option v-for="opcja in opcje" v-bind:value="opcja.value">
    {{ opcja.text }}
  </option>
</select>
<span>Wybrane: {{ wybrane }}</span>
```
``` js
new Vue({
  el: '...',
  data: {
    wybrane: 'A',
    opcje: [
      { text: 'jeden', value: 'A' },
      { text: 'dwa', value: 'B' },
      { text: 'trzy', value: 'C' }
    ]
  }
})
```
{% raw %}
<div id="example-7" class="demo">
  <select v-model="wybrane">
    <option v-for="opcja in opcje" v-bind:value="opcja.value">
      {{ opcja.text }}
    </option>
  </select>
  <span>Wybrane: {{ wybrane }}</span>
</div>
<script>
new Vue({
  el: '#example-7',
  data: {
    wybrane: 'A',
    opcje: [
      { text: 'jeden', value: 'A' },
      { text: 'dwa', value: 'B' },
      { text: 'trzy', value: 'C' }
    ]
  }
})
</script>
{% endraw %}

## Bindowanie wartości

Dla elementów radio, chcebox i select, wartości bindowane przez `v-model` zwykle są statycznymi stringami (lub wartościami logicznymi w przypadku checkbox):

``` html
<!-- `picked`, kiedy jest wybrany przyjmuje wartość "a" -->
<input type="radio" v-model="picked" value="a">

<!-- `toggle` przyjmuje wartość true lub false -->
<input type="checkbox" v-model="toggle">

<!-- `selected` wybrany przyjmuje wartość "abc" -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```

Ale czasami możesz potrzebować bindowania wartości dynamicznej do instancji Vue. Możesz użyć do tego `v-bind`. Ponadto `v-bind` pozwala na bindowanie wartości nie będących łancuchem znaków.

### Checkbox

``` html
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```

``` js
// zaznaczony:
vm.toggle === 'yes'
// nie zaznaczony:
vm.toggle === 'no'
```

<p class="tip">Wartość atrybutu `true` i `false` nie wpływa na atrybut `value`, ponieważ przeglądarka nie wysyła niezaznaczonych checkboxów. Aby wysłać taką wartość (np: "yes" lub "no"), użyj radio butonów.</p>

### Radio butony

``` html
<input type="radio" v-model="wybrany" v-bind:value="a">
```

``` js
// jeżeli wybrany:
vm.wybrany === vm.a
```

### Select Options

``` html
<select v-model="wybrany">
  <!-- obiekt osadzony lokalnie -->
  <option v-bind:value="{ liczba: 123 }">123</option>
</select>
```

``` js
// jeżeli wybrany:
typeof vm.wybrany // => 'object'
vm.wybrany.liczba // => 123
```

## Modyfikatory

### `.lazy`


Domyslnie `v-model` synchronizuje input z obiektem 'data' przy kazdym wystapieniu zdarzenia `input` (za wyjatkiem kompilacji IME [jak wspomniano wcześniej](#vmodel-ime-tip)). Możesz doadać modyfikator `lazy` zamiast synchronizować przy zdarzeniu `change`:

``` html
<!-- synchronizowane po zdarzeniu "change" zamiast "input" -->
<input v-model.lazy="msg" >
```

### `.number`

Jeśli chcesz, aby dane wprowadzane przez użytkownika były automatycznie numerowane, możesz dodać modyfikator `number` do danych wejściowych `v-model`:


``` html
<input v-model.number="age" type="number">
```

To jest bardzo uzyteczne, ponieważ nawet wartość inputa `type="number"` w htmlu zawsze zwraca string.

### `.trim`

Jeśli chcesz, aby dane wprowadzane przez użytkownika były automatycznie przycinane, możesz dodać modyfikator "trim" inputów zarządzanych w `v-model`:

```html
<input v-model.trim="msg">
```

## `v-model` w komponentach

> Jeżeli jeszcze nie znasz komponentów Vue, możesz to na razie pominąć.

Wbudowane w htmla typy inputów nie zawsze odpowiadają naszym potrzebom. Na szczęscie komponenty Vue pozwalają na budowanie inputów z całkowicie personalizowanym zachowaniem, których można wielokrotnie używać. Te inputy działają też z `v-model`! Aby dowiedzieć się więcej, przeczytaj [personalizowa inputy](components.html#Form-Input-Components-using-Custom-Events) w dziale komponenty.
