<template lang='pug'>
  div 
    input.search(v-stream:keyup="{ subject: inputValue$, data: inputValue }") 
    p {{ inputValue }}
    p {{ text }}
    div#el
      input(v-model="search")
      p {{ message$ }}
      div(v-if="results")
        h1 {{ results.term }}
        ul(v-if="results.matches.length")
          li(v-for="match in results.matches")
            a(:href="match.url")
              | {{ match.title }}
            p
              | {{ match.description }}
      p(v-else)
        | No matches found.
</template>

<script lang="ts">
import {
  Component,
  Prop,
  Vue
} from "nuxt-property-decorator"
//import { Observable, interval } from 'rxjs';
//import { filter } from 'rxjs/operators'
import $ from 'jQuery'
import { ReplaySubject, from, timer} from 'rxjs'
import { skipUntil, bufferTime, skip, takeWhile, take, toArray, skipWhile, bufferCount, tap, pluck, windowCount, filter, startWith, scan, debounceTime, mergeScan, distinctUntilChanged, switchMap, map } from 'rxjs/operators'

function fetchTerm (term) {
  return from($.ajax({
    url: 'http://en.wikipedia.org/w/api.php',
    dataType: 'jsonp',
    data: {
      action: 'opensearch',
      format: 'json',
      search: term
    }
  }).promise())
}

function formatResult (res) {
  return {
    term: res[0],
    matches: res[1].map((title, i) => ({
      title: title,
      description: res[2][i],
      url: res[3][i]
    }))
  }
}

const messageObservable = from(
  ['Example Message', 'Example Message Final']
);

@Component<Search>({
  subscriptions () {
    const matcher = new RegExp(/^([a-zA-Z]{1})$/)
    return {
      inputValue: this.$fromDOMEvent('input','keyup').pipe(
        pluck('key'),
        filter((n: any) => n.match(matcher)),
        bufferTime(2000),
        filter((n: any) => n.length > 0),
        distinctUntilChanged(),
        tap(n => console.log(n)),
        //tap(val => console.log(val)),
      ),
      text: this.inputValue$.pipe(
        skip(1),
        tap(n => console.log(n)),
        pluck('data'),
        filter((n: any) => n !== undefined),
        distinctUntilChanged(),
        bufferCount(6, 1),
        //tap(n => console.log(n))
        //map(key => {
        //    console.log(key + '2333333333')
        //}),
      ),
      //.pluck('target', 'value'),
      message$: messageObservable,
      results: this.$watchAsObservable('search').pipe(
        pluck('newValue'),
        //filter((text) => {
        //  //text.length > 2
        //}),
        debounceTime(500),
        distinctUntilChanged(),
        switchMap(fetchTerm),
        map(formatResult)
      ),
      //music: this.inputValue$.pipe(
      //  pluck('data'),
      //  map(data => {
      //      console.log('aaaaaaaa' + data)
      //  }),
      //)
      //count: this.plus$.subscribe(
      //  value => console.log(value),
      //)
    }
  }
})

export default class Search extends Vue {
  @Prop() person
  el: '$el'
  key: string = ""
  search: string = ""
  inputValue$ = new ReplaySubject(3)
  text: string = ""
  music: string = ""

  //created() {
  //  this.$watchAsObservable('search')
  //    .interval(1000)
  //    .filter((value) => value % 2 === 0)
  //    .subscribe((value) => { this.count = value })
  //}
}

</script>
<style scoped>
.search {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana,
    sans-serif;
  padding: 1rem;
  margin: 0.25rem;
  border: 0.25rem solid gainsboro;
  height: 30px;
  width: 100%
}
</style>
