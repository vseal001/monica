<template>
  <div>
    <p v-if="title" class="mb2" :class="{ b: required }">
      {{ title }}
    </p>
    <input type="hidden" :name="name" :value="selected ? selected.id : ''" />
    <v-select
      v-model="selected"
      :placeholder="placeholder"
      :label="'complete_name'"
      :options="items"
      @search="search"
      @search:blur="blur"
    />
  </div>
</template>

<script>
import vSelect from 'vue-select';
import 'vue-select/dist/vue-select.css';
import axios from 'axios';

export default {
  components: {
    vSelect
  },
  props: {
    name: {
      type: String,
      default: '',
    },
    title: {
      type: String,
      default: '',
    },
    required: {
      type: Boolean,
      default: true,
    },
    userContactId: {
      type: String,
      default: '',
    },
    defaultOptions: {
      type: Array,
      default: () => [],
    },
    placeholder: {
      type: String,
      default: '',
    },
    wait : {
      type: Number,
      default: 200,
    },
  },

  data () {
    return {
      selected: null,
      items: [],
      callUpdateItems: null,
      cache: [],
    };
  },

  mounted() {
    this.callUpdateItems = _.debounce((text) => {
      this.getContacts(text, this)
        .then((response) => {
          this.cache[text] = response;
          this.displayItems(text);
        });
    }, this.wait);
    this.items = this.defaultOptions;
  },

  methods: {

    updateItems (text) {
      if (text === null) {
        return;
      }
      if (text.length < this.minLen) {
        this.items = [];
        return;
      }

      if (this.cache[text] === undefined) {
        this.callUpdateItems(text);
      } else {
        this.callUpdateItems.cancel();
        this.displayItems(text);
      }
    },

    displayItems (text) {
      var datas = [];
      if (text === undefined || text.length === 0) {
        datas = this.defaultOptions;
      } else {
        datas = this.cache[text];
      }

      datas = datas.filter(this.filter);

      this.items = datas;
    },

    getContacts: function (keyword, vm) {
      return axios.post('/people/search', {
        needle: keyword
      }).then(function(response) {
        return response.data.data;
      });
    },

    filter(item) {
      return this.userContactId !== item.id;
    },

    search(keyword, loading) {
      this.updateItems(keyword);
    },

    blur (e) {
      this.items = this.defaultOptions;
    },
  }
};
</script>
