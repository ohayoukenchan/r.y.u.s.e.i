<template lang='pug'>
  .drag-and-drop
    h2 DRAG AND DROPPING
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
        | {{ item.text }}
</template>

<script lang="ts">

type Item = {
  text: string,
  index: number,
  selected: boolean,
}

let basicitems: Array<Item> = [{
  text: 'バナナ',
  index: 0,
  selected: false,
},{
  text: 'りんご',
  index: 1,
  selected: false,
},{
  text: 'みかん',
  index: 2,
  selected: false,
},{
  text: 'いちご',
  index: 3,
  selected: false,
},{
  text: 'ぶどう',
  index: 4,
  selected: false,
},{
  text: 'きうい',
  index: 5,
  selected: false,
},{
  text: 'かき',
  index: 6,
  selected: false,
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
  //@Prop()

  items: Array<Item> = basicitems;
  dragging: Boolean = false;
  selectedItems: Array<Item> = [];
  targetIndex: number = 0;

  @Watch('dragging')
  onValueChange(): void {
    console.log('hohe')
    this.selectedItems = this.items.filter(r => { return r.selected });
  }

  mounted() {
    if(process.browser) {
      //const target = <HTMLElement>document.getElementById('draggable')
      //const target = this.$el.getElementsByTagName('ul')
      //alert(target)
      //console.log(target[0])
      //target[0].addEventListener('drop', () => {
      //  this.drop()
      //}, false)
    }
  }
  //@Watch('items')
  //onValueChange(items): void {
  //  console.log(items)
  //}

  ddd(): void {
    alert('ddd')
  }

  onDrop(event): void {
    console.log('dropppping')
    console.log(event)
    if (event.dataTrasfer) {
      const data = event.dataTransfer.getData("text/plain");
      event.target.textContent = data;
      event.preventDefault();
    }
    this.targetIndex = event.target.getAttribute('data-index')
  }

  onSelected(index): void {
    //console.log('selected')
    (event as Event).stopPropagation();
    this.items[index].selected = !this.items[index].selected
  }

  onDragStart(event): void {
    //alert('dragstart')
    console.log('dragstart')
    this.dragging = true;
    if (event.dataTransfer.setDragImage) {
      var img = new Image();
      img.src = 'framework/vendor/ic_content_copy_black_24dp_2x.png';
      event.dataTransfer.setDragImage(img, 0, 0);
    }
  }
  onDragEnd(): Boolean {
    //alert('dragend')
    const index: number = this.targetIndex
    this.dragging = false;
    console.log(this.items)
    this.items = this.items.slice(0, index)
                .concat(this.selectedItems)
                .concat(this.items.slice(index));
    this.items.map((r) => { r.selected = false; });
    this.selectedItems.splice(0, this.selectedItems.length);
    console.log('drafend')
    return true;
  }
  onMoved(): void {
    //alert('moved')
    this.items = this.items.filter(r => { return !r.selected; });
    console.log('moved')
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
    this.items.splice(index, 1)
  }
}
</script>

<style scoped>
.drag-and-drop {
}
.drag-and-drop li {
}

.drag-and-drop ul[dnd-list] {
    min-height: 42px;
    padding-left: 0px;
}

.drag-and-drop ul .placeholder {
    background-color: #ddd;
    display: block;
    min-height: 42px;
}

.drag-and-drop ul li {
    background-color: #fff;
    border: 1px solid #ddd;
    border-top-right-radius: 4px;
    border-top-left-radius: 4px;
    display: block;
    margin-bottom: -1px;
    padding: 10px 15px;
}

.drag-and-drop ul li.selected {
    background-color: #dff0d8;
    color: #3c763d;
}
</style>
