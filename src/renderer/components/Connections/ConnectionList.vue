<template >
  <div v-if="list.length">
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="#">{{ title }}</a>
      <el-button-group
        v-if="delete_select_mode">
        <el-button
          type="secondary"
          icon="el-icon-close"
          @click="cancel_delete">Cancel</el-button>
        <el-button
          type="danger"
          icon="el-icon-delete"
          @click="confirm_delete">Confirm</el-button>
      </el-button-group>
      <el-button-group>
        <el-button
          v-if="!delete_select_mode"
          type="secondary"
          icon="el-icon-delete"
          @click="delete_select_mode = true"></el-button>
        <el-button
          type="primary"
          icon="el-icon-refresh"
          @click="$emit('refresh',)"></el-button>
      </el-button-group>
    </nav>
    <el-collapse v-model="expanded_items">
      <ul class="connection-list">
        <div
          v-for="(connection) in page"
          class="connection-item">
          <el-checkbox
            v-if="delete_select_mode"
            class="delete-check"
            @change="update_mark_for_deletion($event, connection)"></el-checkbox>
          <el-collapse-item
            :name="connection.connection_id"
            :key="title+connection.connection_id"
            :title="get_name(connection)">
            <el-row>
              <div>
                <vue-json-pretty
                  :deep=1
                  :data="connection">
                </vue-json-pretty>
              </div>
              <template v-if="editable">
                <el-button @click="edit(connection)">Edit</el-button>
              </template>
              <el-button type="danger" @click="delete_conn(connection)">Delete</el-button>
              <el-button v-on:click="collapse_expanded(connection)">^</el-button>
            </el-row>
          </el-collapse-item>
        </div>
      </ul>
    </el-collapse>
    <el-pagination
      layout="prev, pager, next"
      :total="list.length"
      :hide-on-single-page="true"
      :page-size="limit"
      @current-change="page_turn">
    </el-pagination>
    <el-dialog title="Edit Connection" :visible.sync="editFormActive">
      <el-form :model="editForm">
        <el-form-item label="Role:" :label-width="formLabelWidth">
          <el-input v-model="editForm.role" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="Label:" :label-width="formLabelWidth">
          <el-input v-model="editForm.label" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editFormActive = false">Cancel</el-button>
        <el-button type="primary" @click="update">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<style>
.connection-list {
  padding: 0 0 0 0;
  margin: 0 1em 0 1em;
}
.connection-list .connection-item {
  display: flex;
  flex-direction: row;
  flex-grow: 1;
}
.connection-item .el-checkbox {
  margin: 1em 8px 0 0;
}
.connection-list .el-collapse-item {
  width: 100%;
}
</style>

<script>
import VueJsonPretty from 'vue-json-pretty';

export default {
  name: 'connection-list',
  props: ['title', 'list','editable'],
  components: {
    VueJsonPretty,
  },
  data () {
    return {
      limit: 15,
      offset: 0,
      delete_select_mode: false,
      marked_for_deletion: new Set(),
      expanded_items:[],
      editFormActive: false,
      editForm: {
        connection_id: '',
        role: '',
        label: '',
      },
      formLabelWidth: '100px'
    }
  },
  computed: {
    page: function() {
      return this.list.slice(this.offset, this.offset + this.limit);
    }
  },
  methods: {
    get_name: function(connection) {
      if('their_label' in connection) {
        return connection.their_label;
      };
      return connection.connection_id;
    },
    edit: function (connection) {
      this.editForm.connection_id = connection.connection_id;
      this.editForm.role = connection.their_role;
      this.editForm.label = connection.their_label;
      this.editFormActive = true;
    },
    update: function() {
      this.editFormActive = false;
      this.$emit('connection-editted', this.editForm);
    },
    delete_conn: function (connection) {
      this.$emit('connection-deleted', connection);
    },
    collapse_expanded: function(connection){
      this.expanded_items = this.expanded_items.filter(
        item => item != connection.connection_id
      );
    },
    page_turn: function(new_page) {
      this.offset = (new_page - 1) * this.limit;
    },
    update_mark_for_deletion: function(mark, connection) {
      if (mark) {
        this.marked_for_deletion.add(connection.connection_id);
      } else {
        this.marked_for_deletion.delete(connection.connection_id);
      }
    },
    confirm_delete: function() {
      this.delete_select_mode = false;
      this.$emit('connection-delete-many', this.marked_for_deletion);
      this.marked_for_deletion.clear();
    },
    cancel_delete: function() {
      this.delete_select_mode = false;
      this.marked_for_deletion.clear();
    }
  }
}
</script>
