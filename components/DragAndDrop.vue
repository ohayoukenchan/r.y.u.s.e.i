<template lang='pug'>
  div
    .header
      .header-title
        a.link
          svg.icon.icon-right
             use(xlink:href="/images/sprite.svg#right")
          | 記事の編集に戻る
        h1.title
          | ゲーミングヘッドセットおすすめランキング20選※記事タイトル
        .submit
          button(@click="onSubmit" type="submit")
            | 変更を保存
      div(v-if="selectedItems.length > 0")
        | {{ selectedItems.length }}
        | 件選択中
      button(@click="insertMultipleItems(0)")
        | ▲
      button(@click="insertMultipleItems(items.length)")
        | ▼
      button(@click="selectAllItems")
        | ぜん選択
      button(@click="deleteSelectedItems")
        | さくじょ

    //img(src='/images/ic_content_copy_black_24dp_2x.png')
    .drag-and-drop
      ul#draggable(@dragover.prevent, @drop="onDrop")
        transition-group(name="fade" duration="2400")
          li(v-for="(item, index) in items",
            :class="{ selected: item.selected }",
            :key="item.partsCode",
            draggable="true",
            @click="onSelected(index)",
            @dragstart="onDragStart",
            @dragend="onDragEnd(index)",
            @dragleave="onMoved",
            :data-index="index"
          )
            .button-group
              button(@click="insertTop(index)")
                svg.icon.icon-top
                  use(xlink:href="/images/sprite.svg#top")
              button(@click="insertTop(index)")
               svg.icon.icon-prev
                  use(xlink:href="/images/sprite.svg#prev")
              button(@click="insertAfter(index)")
                svg.icon.icon-next
                  use(xlink:href="/images/sprite.svg#next")
              button(@click="insertBottom(index)")
                svg.icon.icon-bottom
                  use(xlink:href="/images/sprite.svg#bottom")
              button.button-trash(@click="deleteItem(index)")
                svg.icon.icon-trash
                  use(xlink:href="/images/sprite.svg#trash")
            svg.icon.icon-grab
              use(xlink:href="/images/sprite.svg#grab")
            .checkbox(:class="selectedItems.length > 1 ? 'multiple' : ''")
              span
            span.label
              | {{ item.type }}
            img(v-if="item.image !== ''" :src='item.image' width="50" height="50")
            p
              | {{ item.text }}
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
  text: 'ぶたがつぶたにちぶたようびきょうそらからぶたがふってきて',
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
},{
  text: '商品詳細のリリースお疲れ様でした。納期が迫る中最低限の実装で効果を試せるよう',
  partsCode: 7,
  selected: false,
  type: '商品',
  image : '',
},{
  text: '平面のデザインにとどまらず立体の方へ突破していく鈴木さんに新しい風',
  partsCode: 8,
  selected: false,
  type: '商品',
  image : '',
}





]

interface HTMLElementEvent<T extends HTMLElement> extends Event {
  target: T;
}

import objectAssignDeep from 'object-assign-deep';
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
  selectedItems: Array<Item> = [];
  selectedIndex: number;
  targetIndex: number = 0;

  onSelected(index): void {
    (event as Event).stopPropagation();
    if((event as any).shiftKey) {
      const targetIndex:number = parseInt((event as any).target.getAttribute('data-index'))
      const startIndex = this.selectedIndex > targetIndex ? targetIndex : this.selectedIndex;
      const endIndex:number = this.selectedIndex > targetIndex ? this.selectedIndex : targetIndex;
      this.items.map((r, index) => { if(startIndex <= index && endIndex >= index) { r.selected = true }});
    } else {
      this.items[index].selected = !this.items[index].selected;
    }
    this.selectedIndex = index;
    this.selectedItems = this.items.filter(r => { return r.selected });
  }

  onDragStart(event): void {
    this.dragging = true;
    this.items = this.items.filter(r => { return !r.selected; });
    if (event.dataTransfer.setDragImage) {
      let image = new Image();
      image.src = '/images/ic_content_copy_black_24dp_2x.png';
      event.dataTransfer.setDragImage(image, 0, 0);
    }
  }

  onMoved(event): void {
    (event as Event).stopPropagation();
    const insertItem = event.target;
    const insertIndex = insertItem.getAttribute('data-index')
    if (insertIndex !== null ) {
      this.removePlaceholder();
      const rect = insertItem.getBoundingClientRect();
      const isFirstHalf = event.clientY < rect.top + rect.height / 2;
      const li = document.createElement('li')
      const ul = insertItem.parentElement
      console.log(isFirstHalf)
      console.log(event.clientYu)

      li.classList.add('placeholder')
      ul.insertBefore(li, isFirstHalf ? insertItem : insertItem.nextSibling);
      this.targetIndex = isFirstHalf ? insertIndex - 1 : insertIndex
    }
  }
  onDrop(event): void {
    if (event.dataTrasfer) {
      const data = event.dataTransfer.getData("text/plain");
      event.target.textContent = data;
      event.preventDefault();
    }
  }
  onDragEnd(): Boolean {
    this.removePlaceholder();
    if(!this.dragging || this.selectedItems.length <= 0) {
      this.dragging = false;
      return false;
    }
    let index:number = Number(this.targetIndex);
    const insertIndex = index += 1;
    this.insertItems(insertIndex);
    this.resetSelection();
    return true;
  }
  removePlaceholder(): void {
    const placeholder = document.querySelector('.placeholder')
    if(placeholder) {
      placeholder.remove();
    }
  }
  insertMultipleItems(index): void {
    this.items = this.items.filter(r => { return !r.selected; });
    this.insertItems(index);
    this.resetSelection();
  }
  resetSelection(): void {
    this.items.map((r) => { r.selected = false; });
    this.selectedItems.splice(0, this.selectedItems.length);
    this.dragging = false;
  }
  insertItems(index): void {
    this.items = this.items.slice(0, index)
                .concat(this.selectedItems)
                .concat(this.items.slice(index));
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
    this.items = this.items.filter(r => { return !r.selected })
  }
  onSubmit(): void {
    const partsCodes: Array<Number> = this.items.map(r => { return r.partsCode })
    alert(partsCodes)
  }
}
</script>

<style lang='scss'>
.drag-and-drop {
  width: 1020px;
  margin: 0 auto;
}

.drag-and-drop .label {
  font-size: 8px;
  color: #fff;
  background-color: #111;
  border-radius: 2px;
  padding: 3px;
  width: 53px;
  margin-right: 10px;
  text-align: center;
}

.drag-and-drop ul {
  min-height: 42px;
  padding-left: 0px;
  border: 1px solid rgba(100,121,143,0.122);
}

.drag-and-drop ul li {
  align-items: center;
  background-color: #fff;
  box-shadow: inset 0 -1px 0 0 rgba(100,121,143,0.122);
  cursor: move;
  display: flex;
  padding: 10px 15px;
  position: relative;

  &:hover {
    margin-bottom: 3px;
    box-shadow: inset 1px 0 0 #dadce0, inset -1px 0 0 #dadce0, 0 1px 2px 0 rgba(60,64,67,.3), 0 1px 3px 1px rgba(60,64,67,.15)
  }

  p {
    margin-left: 10px;
  }

}

.button-group {
  button {
    appearance: none;
    background: #f7f7f7;
    border-radius: 2px;
    border: none;
    margin-right: 9px;
    padding: 5px 13px;
    
    &:last-child {
      margin-right: 0;
    }

    &:hover {
      background: #e7e7e7;

      .icon {
        fill: #999;
      }

      .icon-trash {
        fill: #fff;
      }
    }

    &.button-trash {
      background: #DA3D3D;

      &:hover {
        background: #B91313;
      }
    }
  }
}

.icon {
  display: inline-block;
  fill: #c7c7c7;
  width: 14px;
  height: 14px;
  vertical-align: text-bottom;

  &.icon-grab {
    width: 6px;
    height: 12px;
    margin-right: 10px;
  }

  &.icon-top {
  }

  &.icon-trash {
    fill: #fff;
  }
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
  padding: 0;
  position: relative;
  height: 2px;
  background: #02BB80;
  
  &:before {
    display: block;
    padding: 15px 23px;
    position: absolute;
    right: -57px;
    bottom: -48px;
    border: 2px solid #02BB80;
    border-radius: 2px;
    background: #E5F8F2;
    color: #02BB80;
    content: 'パーツを移動';
    font-weight: bold;
    z-index: 3000;
  }
}

.drag-and-drop ul li.selected,
.drag-and-drop ul li.fade-enter-active {
  background-color: rgba(2, 187, 128, .1);
}

.drag-and-drop {
  .checkbox {
    display: inline-block;
    color: #fff;
    cursor: pointer;
    position: relative;

    span {
      display: inline-block;
      position: relative;
      background-color: transparent;
      width: 20px;
      height: 20px;
      transform-origin: center;
      border: 2px solid #e7e7e7;
      border-radius: 2px;
      vertical-align: -6px;
      margin-right: 10px;
      //transition: background-color 100ms 100ms, transform 350ms cubic-bezier(.78,-1.22,.17,1.89);

      &:before {
        content: "";
        width: 0px;
        height: 2px;
        border-radius: 2px;
        background: #fff;
        position: absolute;
        transform: rotate(45deg);
        top: 9px;
        left: 5px;
        transition: width 50ms ease 50ms;
        transform-origin: 0% 0%;
      }

      &:after {
        content: "";
        width: 0;
        height: 2px;
        border-radius: 2px;
        background: #fff;
        position: absolute;
        transform: rotate(305deg);
        top: 14px;
        left: 7px;
        transition: width 50ms ease;
        transform-origin: 0% 0%;
      }
    }

    &:hover {
      span {
        &:before {
          width: 5px;
          transition: width 100ms ease;
        }

        &:after {
          width: 10px;
          transition: width 150ms ease 100ms;
        }
      }
    }
  }
}
.drag-and-drop li {

  &.selected {
    .checkbox {
      span {
        border-color: #02BB80;
        background-color: #02BB80;
        transform: scale(1.05);

        &:after {
          width: 12px;
          background: #fff;
        }

        &:before {
          width: 5px;
          background: #fff;
        }

        &:hover {
          span {
            background-color: #fff;
            transform: scale(1.05);

            &:after {
              width: 10px;
              background: #1790b5;
              transition: width 150ms ease 50ms;
            }

            &:before {
              width: 5px;
              background: #1790b5;
              transition: width 150ms ease 50ms;
            }
          }
        }
      }
    }

    //.multiple {
    //  &.checkbox {
    //    span {
    //      &:after {
    //        transform: rotate(180deg);
    //        top: 12px;
    //        left: 16px;
    //      }
    //      &:before {
    //        transform: rotate(180deg);
    //        top: 12px;
    //        left: 16px;
    //      }
    //    }
    //  }
    //}
  }
}

.header {

  .header-title {

    display: flex;
    justify-content: center;
    position: relative;
    padding: 10px;
    .title {
      font-size: 15px;
      text-align: center;
    }
    .link {
      left: 20px;
      position: absolute;
      cursor: pointer;

      .icon-right {
        margin-right: 10px;
      }
    }
    .submit {
      right: 20px;
      position: absolute;
      button {
        appearance: none;
        border: 1px solid #666;
        color: #666;
        cursor: pointer;
        border-radius: 2px;
        background-color: transparent;
      }
    }
    color: #fff;
    background: #333;
  }
}


</style>
