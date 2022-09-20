<template>
  <div class="fluid container">
    <div class="row">
      <div class="form-group form-group-lg panel panel-default">
        <div class="panel-heading">
          <h3 class="panel-title">Menu Items</h3>
        </div>
        <div class="panel-body w-full">
          <div class="md:flex md:items-center mb-6">
            <div class="md:w-2/3">
            <label for="newNavItem" class="block text-gray-500 font-bold mb-1 md:mb-0 pr-4">
                Nav Item <small style="color: red;" v-html="newNavItemError"></small>
              </label>
              <input v-model="newNavItem" type="text" @keyup.enter="addNewNavItem" id="newNavItem" class="bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500">
            </div>
            <div class="md:w-1/3">
              <button type="button" @click="addNewNavItem" class="inline-flex items-center ml-5 mt-8 px-4 py-2 bg-gray-800 border border-transparent rounded-md font-semibold text-base text-white uppercase tracking-widest hover:bg-gray-700 active:bg-gray-900 focus:outline-none focus:border-gray-900 focus:ring focus:ring-gray-300 disabled:opacity-25 transition">Add new nav item</button>
            </div>
          </div>
        </div>
      </div>

      <!-- TODO: Delete navItems, and make new links look good -->

      <draggable class="list-group child" tag="span" v-model="nav" v-bind="dragOptionsParent" :move="onMove" @start="isDragging=true" @end="isDragging=false">
        <transition-group class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 sm:gap-4 p-4" type="transition" :name="'navs'" tag="div">
        
          <!-- Menu -->
          <div class="list-group-item parent" v-for="(subnav, navIndex) in nav" :key="navIndex">
            <i class="fa-regular fa-folder-open"></i> {{ nav[navIndex].title }}

            <h2>New Link</h2>

            <div v-if="nav[navIndex].links.length > 10">
              <div class="bg-orange-100 border-l-4 border-orange-500 text-orange-700 p-4" role="alert">
                <p class="font-bold">Woah. This is starting to get big...</p>
                <p>You might confuse and frustrate users with too many links in your menus. Can you simplify it?</p>
              </div>
            </div>

            <details>
              <summary>Open New Link Section</summary>

              <div style="color: red;" :v-if="nav[navIndex].newLink.error.length > 0">
                {{ nav[navIndex].newLink['error'] }}
              </div>

              <label :for="'linkTitle-'+navIndex">Link Title</label>
              <input :id="'linkTitle-'+navIndex" @keyup.enter="addNewLink(navIndex)" v-model="nav[navIndex].newLink['title']" type="text" class="bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500">

              <label :for="'linkHref-'+navIndex">URL</label>
              <input :id="'linkHref-'+navIndex" @keyup.enter="addNewLink(navIndex)" v-model="nav[navIndex].newLink['href']" type="text" class="bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500">

              <input :id="'linkNewTab-'+navIndex" @keyup.enter="addNewLink(navIndex)" v-model="nav[navIndex].newLink['target']" type="checkbox" class="leading-tight">
              <label :for="'linkNewTab-'+navIndex" class="ml-2">Open in new tab</label>
              <div>
                <button @click="addNewLink(navIndex)" class="inline-flex items-center px-4 py-2 bg-gray-800 border border-transparent rounded-md font-semibold text-base text-white uppercase tracking-widest hover:bg-gray-700 active:bg-gray-900 focus:outline-none focus:border-gray-900 focus:ring focus:ring-gray-300 disabled:opacity-25 transition">Add new link</button>
              </div>
            </details>

            <draggable class="list-group child" tag="span" v-model="nav[navIndex].links" v-bind="dragOptionsChild" :move="onMove" @start="isDragging=true" @end="isDragging=false">
              <transition-group type="transition" :name="'nav-list'" tag="ul">
                <li class="list-group-item" v-for="item in nav[navIndex].links" :key="item.id">
                  <i class="fa-solid fa-grip-vertical mr-3"></i>
                  {{item.title}} 
                  <a :href="item.href" target="_blank" class="inline-flex items-center px-4 py-2 bg-gray-800 border border-transparent rounded-md font-semibold text-base text-white uppercase tracking-widest hover:bg-gray-700 active:bg-gray-900 focus:outline-none focus:border-gray-900 focus:ring focus:ring-gray-300 focus:text-white hover:text-white disabled:opacity-25 transition">Open</a>
                  <i class="fa-solid fa-trash" @click="deleteLink(navIndex, item.id)"></i>
                  <!-- <span class="badge">{{item.id}}</span> -->
                </li>
              </transition-group>
            </draggable>
          </div>
          <!-- End Menu -->

        </transition-group>
      </draggable>

    </div>
    <div class="row mt-10">
      <pre>{{navString}}</pre>
    </div>
  </div>
</template>

<script>
import draggable from "vuedraggable";
const message = [
  "vue.draggable",
  "draggable",
  "component",
  "for",
  "vue.js 2.0",
  "based",
  "on",
  "Sortablejs"
];

export default {
  name: "hello",
  components: {
    draggable
  },
  data() {
    return {
      newNavItem: "",
      newNavItemError: "",
      nav: [],
      isDragging: false,
      delayedDragging: false
    };
  },
  methods: {
    onMove({ relatedContext, draggedContext }) {
      const relatedElement = relatedContext.element;
      const draggedElement = draggedContext.element;
      return (
        (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
      );
    },
    slugify(str) {
      return str.toLowerCase().trim().replace(/[^\w\s-]/g, '').replace(/[\s_-]+/g, '-').replace(/^-+|-+$/g, '');
    },
    uuidv4() {
      return ([1e7]+-1e3+-4e3+-8e3+-1e11).replace(/[018]/g, c =>
        (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
      );
    },
    navItemsAlreadyContains(itemTitle) {
      let found = false;
      this.nav.forEach(item => {
        if (item.title == itemTitle) {
          found = true;
        }
      });
      return found;
    },
    addNewNavItem() {
      if (this.newNavItem == "") return;
      if (this.navItemsAlreadyContains(this.newNavItem)) {
        this.newNavItemError = "This item already exists.";
        return;
      } else {
        this.newNavItemError = "";
      }
      
      let id = this.slugify(this.newNavItem);

      this.nav.push({
        title: this.newNavItem,
        id: this.uuidv4(),
        fixed: false,
        links: [],
        newLink: {
          title: "",
          id: "",
          href: "",
          target: true,
          error: ""
        }
      });
      this.newNavItem = "";

      // completely replace the object so view picks it up
      // this.nav = {
        // ...this.nav
      // };

    },
    addNewLink(navItemId) {

      let link = this.nav[navItemId].newLink;

      if (link.title == "" || link.href == "") {
        if (link.title == "" && link.href == "") { link.error = "Title and URL are required"}
        else if (link.title == "") { link.error = "Title is required"}
        else { link.error = "URL is required"}
        return;
      }
      link.error = "";

      // let title = "entry #" + Math.floor(Math.random() * 99999999);
      let id = this.uuidv4();
      let linkToAdd = {
        title: link.title,
        id: id,
        href: link.href,
        target: link.target ? "_blank" : "",
        fixed: false,
        error: ""
      };
      this.nav[navItemId].links.push(linkToAdd);
      // clear the form
      this.nav[navItemId].newLink = {
        title: "",
        id: "",
        href: "",
        target: true,
        error: ""
      };
    },
    deleteLink(navItemId, linkId) {
      let links = this.nav[navItemId].links;
      for (let i = 0; i < links.length; i++) {
        if (links[i].id == linkId) {
          links.splice(i, 1);
          break;
        }
      }
    }
  },
  computed: {
    dragOptionsParent() {
      return {
        animation: 200,
        group: "parent",
        disabled: false,
        ghostClass: "ghost",
        filter: ".no-drag"
      };
    },
    dragOptionsChild() {
      return {
        animation: 200,
        group: "child",
        disabled: false,
        ghostClass: "ghost",
        filter: ".no-drag"
      };
    },
    navString() {
      let deepCopy = JSON.parse(JSON.stringify(this.nav));
      let nav = deepCopy.map(item => {
        delete item.newLink;
        delete item.id;
        delete item.fixed;
        item.links.forEach((link) => {
          delete link.id;
          delete link.fixed;
          delete link.error;
        });
        return item;
      });
      
      return JSON.stringify(nav, null, 2);
    }
  },
  watch: {
    isDragging(newValue) {
      if (newValue) {
        this.delayedDragging = true;
        return;
      }
      this.$nextTick(() => {
        this.delayedDragging = false;
      });
    }
  }
};
</script>

<style>
.flip-list-move {
  transition: transform 0.5s;
}

.no-move {
  transition: transform 0s;
}

.ghost {
  opacity: 0.5;
  background: #c8ebfb;
}


.list-group {
  display: block;
  min-height: 50px;
  border-radius: 8px;
  border: 2px dashed rgb(0, 135, 247);
  margin-top: 10px;
}

.list-group ul {
  padding: 0 10px;
  min-height: 50px;
  margin-bottom: 0;
}

.list-group ul:empty:after {
  content: "Add or drag links here";
  padding: 15px;
  text-align: center;
  display: block;
}

.list-group.child ul {
  padding: 10px;
}

.list-group-item.parent {
  margin-top: 0;
}

.list-group-item {
  cursor: move;
}

.list-group-item.child:first-child {
  margin-top: 10px;
}

.list-group-item.child:last-child {
  margin-bottom: 10px !important;
}

.list-group-item i {
  cursor: pointer;
}
</style>
