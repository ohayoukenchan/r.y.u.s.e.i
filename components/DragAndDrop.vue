<template lang='pug'>
  div
    div
      div(v-if="selectedItems.length > 0")
        | {{ selectedItems.length }}
        | 件選択中
      button(@click="selectAllItems")
        | ぜん選択
      button(@click="deleteSelectedItems")
        | さくじょ
    img(src='/images/ic_content_copy_black_24dp_2x.png')
    .drag-and-drop
      h2 DnDのデモ
      ul#draggable(@dragover.prevent, @drop="onDrop")
        li(v-for="(item, index) in items",
          :class="{ selected: item.selected }"
          draggable="true",
          @click="onSelected(index)",
          @dragstart="onDragStart",
          @dragend="onDragEnd(index)",
          @dragleave="onMoved",
          :data-index="index"
        )
          .button-group
            button(@click="insertBefore(index)")
              | ▲
            button(@click="insertAfter(index)")
              | ▼
            button(@click="insertTop(index)")
              | ∧
            button(@click="insertBottom(index)")
              | ∨
            button(@click="deleteItem(index)")
              | 削除
          span.label
            | {{ item.type }}
          img(v-if="item.image !== ''" :src='item.image' width="40" height="40")
          | {{ item.text }}
    button(@click="onSubmit" type="submit")
      | 送信
</template>

<script lang="ts">

type Item = {
  text: string,
  partsCode: number,
  selected: boolean,
  type: string,
  image: string
}

let basicitems: Array<Item> = [{
  text: '一粒で幸せ！ゴディバのチョコの魅力とは？',
  partsCode: 0,
  selected: false,
  type: '大見出し',
  image: ''
},{
  text: '日本には、1972年に登場。以来、滑らかな舌触りと芳醇な香り、風味豊かな味わいから',
  partsCode: 1,
  selected: false,
  type: 'テキスト',
  image: ''
},{
  text: 'まずはコレクションに注目！贈る相手や自分の好みにあわせて',
  partsCode: 2,
  selected: false,
  type: '中見出し',
  image: ''
},{
  text: 'ヘーゼルナッツやアーモンドの風味豊かなプラリネが大好き！という方には',
  partsCode: 3,
  selected: false,
  type: 'テキスト',
  image: ''
},{
  text: 'カレアソートメントは四角い板状の形をした、一口サイズのチョコレートのセットです。',
  partsCode: 4,
  selected: false,
  type: '大見出し',
  image: ''
},{
  text: 'ゴディバのバレンタインチョコおすすめ人気ランキング10選【2018年限定品も！】',
  partsCode: 5,
  selected: false,
  type: '大見出し',
  image: ''
},{
  text: '小さなパール状のチョコレート。気軽なおやつにぴったり',
  partsCode: 6,
  selected: false,
  type: '商品',
  image : '/images/aaa.jpg',
}]

interface HTMLElementEvent<T extends HTMLElement> extends Event {
  target: T;
}

import {
  Component,
  Prop,
  Watch,
  Vue
} from "nuxt-property-decorator"



@Component({})

export default class DragAndDrop extends Vue {

  items: Array<Item> = basicitems;
  dragging: Boolean = false;
  passMoved: Boolean = false;
  selectedItems: Array<Item> = [];
  targetIndex: number = 0;

  //@Watch('dragging')
  //OnValueChange(): void {
  //  this.selectedItems = this.items.filter(r => { return r.selected });
  //}

  onSelected(index): void {
    (event as Event).stopPropagation();
    if((event as any).shiftKey) {
      alert('shift key')
    }
    this.items[index].selected = !this.items[index].selected
    this.selectedItems = this.items.filter(r => { return r.selected });
  }

  onDragStart(event): void {
    console.log('1')
    this.dragging = true;
    this.passMoved = false;
    if (event.dataTransfer.setDragImage) {
      let image = new Image();
      image.src = '/images/ic_content_copy_black_24dp_2x.png';
      event.dataTransfer.setDragImage(image, 0, 0);
    }
  }
  onMoved(event): void {
    this.passMoved = true;
    (event as Event).stopPropagation();
    console.log('2')
    this.items = this.items.filter(r => { return !r.selected; });
    const insertItem = event.target;
    const insertIndex = insertItem.getAttribute('data-index')
    if (insertIndex !== null ) {
      const placeholder = document.querySelector('.placeholder')
      if(placeholder) {
        placeholder.remove();
      }
      var rect = insertItem.getBoundingClientRect();
      var isFirstHalf = event.clientY < rect.top + rect.height / 2;
      const li = document.createElement('li')
      li.classList.add('placeholder')
      const ul = insertItem.parentElement
      ul.insertBefore(li, isFirstHalf ? insertItem : insertItem.nextSibling);
      this.targetIndex = isFirstHalf ? insertIndex - 1 : insertIndex
    }
  }
  onDrop(event): void {
    console.log('3')
    if (event.dataTrasfer) {
      const data = event.dataTransfer.getData("text/plain");
      event.target.textContent = data;
      event.preventDefault();
    }
  }
  onDragEnd(): Boolean {
    console.log('4')
    const placeholder = document.querySelector('.placeholder')
    if(placeholder) {
      placeholder.remove();
    }
    if(!this.passMoved || this.selectedItems.length <= 0) {
      this.passMoved = false;
      this.dragging = false;
      return false;
    }
    let index:number = Number(this.targetIndex);
    const insertIndex = index += 1;
    this.items = this.items.slice(0, insertIndex)
                .concat(this.selectedItems)
                .concat(this.items.slice(insertIndex));
    this.items.map((r) => { r.selected = false; });
    this.selectedItems.splice(0, this.selectedItems.length);
    this.passMoved = false;
    this.dragging = false;
    return true;
  }
  insertBefore(index): void {
    (event as Event).stopPropagation();
    this.items.splice(index-1, 2, this.items[index], this.items[index - 1])
  }
  insertAfter(index): void {
    (event as Event).stopPropagation();
    this.items.splice(index, 2, this.items[index+1], this.items[index])
  }
  insertTop(index): void {
    (event as Event).stopPropagation();
    const insertItem = this.items[index]
    this.items.splice(index, 1)
    this.items.unshift(insertItem)
  }
  insertBottom(index): void {
    (event as Event).stopPropagation();
    const insertItem = this.items[index]
    this.items.splice(index, 1)
    this.items.push(insertItem)
  }
  deleteItem(index): void {
    (event as Event).stopPropagation();
    this.items.splice(index, 1)
  }
  selectAllItems(): void {
    let selectedCount: number = 0
    this.items.filter((r) => {
      if(r.selected === true) { selectedCount += 1 };
    });
    if(this.items.length === selectedCount) {
      this.items.map(r => { r.selected = false });
      this.selectedItems = this.items.filter(r => { return r.selected });
      return;
    }
    this.items.map(r => { r.selected = true })
    this.selectedItems = this.items.filter(r => { return r.selected });
  }
  deleteSelectedItems(): void {
    (event as Event).stopPropagation();
    //console.log(this.items)
    this.items = this.items.filter(r => { return !r.selected })
  }
  onSubmit(): void {
    const partsCodes: Array<Number> = this.items.map(r => { return r.partsCode })
    alert(partsCodes)
  }
}
</script>

<style>
.drag-and-drop {
}

.drag-and-drop span {
  font-size: 10px;
  color: #fff;
  background-color: #666;
  border-radius: 4px;
  padding: 3px;
}

.drag-and-drop li {
  position: relative;
}

.drag-and-drop ul {
  min-height: 42px;
  padding-left: 0px;
  border: 1px solid rgba(100,121,143,0.122);
}

.drag-and-drop ul li {
  background-color: #fff;
  box-shadow: inset 0 -1px 0 0 rgba(100,121,143,0.122);
  border-top-right-radius: 4px;
  border-top-left-radius: 4px;
  display: block;
  margin-bottom: 3px;
  padding: 10px 15px;
}

.drag-and-drop ul li:hover {
  box-shadow: inset 1px 0 0 #dadce0, inset -1px 0 0 #dadce0, 0 1px 2px 0 rgba(60,64,67,.3), 0 1px 3px 1px rgba(60,64,67,.15)
}

.drag-and-drop ul li .button-group {
  display: none;
  position: absolute;
  top: 5x;
  right: 10px;
}

.drag-and-drop ul li:hover .button-group {
  display: block;
}


.drag-and-drop ul li.placeholder {
  background-color: #ddd;
  display: block;
  min-height: 42px;
}

.drag-and-drop ul li.selected {
  background-color: #dff0d8;
  color: #3c763d;
}
</style>
