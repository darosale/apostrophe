<template>
  <AposModal
    :modal="modal" modal-title="Manage Page Tree"
    @esc="cancel" @no-modal="$emit('safe-close')"
    @inactive="modal.active = false" @show-modal="modal.showModal = true"
  >
    <template v-if="relationshipField" #primaryControls>
      <AposButton
        type="default" label="Cancel"
        @click="cancel"
      />
      <AposButton
        :label="`Save`" type="primary"
        :disabled="relationshipErrors === 'min'"
        @click="saveRelationship"
      />
    </template>
    <template v-else #primaryControls>
      <AposButton
        type="default" label="Finished"
        @click="cancel"
      />
    </template>
    <template #main>
      <AposModalBody>
        <template #bodyHeader>
          <AposDocsManagerToolbar
            :selected-state="selectAllState"
            @select-click="selectAll"
            @trash-click="trashClick"
            :options="{
              noSearch: true,
              noPager: true
            }"
          />
        </template>
        <template #bodyMain>
          <AposTree
            :items="treeFormatted"
            :headers="headers" :icons="icons"
            v-model="checked"
            :options="treeOptions"
            @update="update" @busy="setBusy"
            @open="openEditor"
          />
        </template>
      </AposModalBody>
    </template>
  </AposModal>
</template>

<script>
import AposModalParentMixin from 'Modules/@apostrophecms/modal/mixins/AposModalParentMixin';
import AposDocsManagerMixin from 'Modules/@apostrophecms/modal/mixins/AposDocsManagerMixin';
import klona from 'klona';

export default {
  name: 'AposPagesManager',
  mixins: [ AposModalParentMixin, AposDocsManagerMixin ],
  emits: [ 'trash', 'search', 'safe-close' ],
  data() {
    return {
      modal: {
        active: false,
        type: 'slide',
        showModal: false
      },
      pageTree: [],
      items: [],
      checked: [],
      // TODO: Get columns from module's browser data.
      options: {
        columns: [
          {
            label: 'Page Title',
            name: 'title'
          },
          {
            label: 'Published',
            name: 'published',
            labelIcon: 'circle',
            icon: 'circle'
          },
          {
            label: 'Edit',
            name: '_id',
            icon: 'pencil',
            iconOnly: true,
            type: 'button',
            action: 'edit'
          },
          {
            label: 'Link',
            name: '_url',
            icon: 'link',
            iconOnly: true,
            type: 'link'
          }
        ]
      }
    };
  },
  computed: {
    treeOptions() {
      return {
        bulkSelect: true,
        draggable: true,
        disableUnchecked: this.relationshipErrors === 'max'
      };
    },
    treeFormatted() {
      const formatted = [];
      if (!this.pageTree || !this.headers.length) {
        return [];
      }

      const pagesSet = klona(this.pageTree);

      pagesSet.forEach(page => {
        const data = {};

        this.headers.forEach(column => {
          data[column.name] = page[column.name];
          data._id = page._id;
          data.children = page.children;
        });
        formatted.push(data);
      });

      return formatted;
    }
  },
  async mounted() {
    // Get the data. This will be more complex in actuality.
    this.modal.active = true;

    await this.getPages();
  },
  methods: {
    async getPages () {
      this.pageTree = [];
      this.items = [];
      const self = this;

      const results = (await apos.http.get(
        '/api/v1/@apostrophecms/page', {
          busy: true,
          qs: {
            all: 1
          }
        }
      )).results;

      formatPage(results);

      this.pageTree = [ results ];

      function formatPage(page) {
        page.published = page.published ? 'Published' : 'Unpublished';
        self.items.push({
          title: page.title,
          _id: page._id
        });

        page.children = page._children;
        delete page._children;

        if (Array.isArray(page.children)) {
          page.children.forEach(formatPage);
        }
      }
    },
    update(obj) {
      // We'll hit a route here to update the docs.
      console.info('CHANGED ROW:', obj);
    },
    setBusy(val) {
      apos.bus.$emit('busy', val);
    },
    trashClick() {
      // TODO: Trigger a confirmation modal and execute the deletion.
      this.$emit('trash', this.selected);
    },
    openEditor(page) {
      console.info('📝 EDIT PAGE', page);
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
