<template>
  <el-row>
    <connection-list
      title="Active Connections:"
      editable="true"
      :list="active_connections"
      @connection-editted="update_connection"
      @connection-deleted="delete_connection"
      @connection-delete-many="delete_connection_many"
      @refresh="fetch_connections"></connection-list>
    <connection-list
      title="Pending Connections:"
      editable="true"
      :list="pending_connections"
      @connection-editted="update_connection"
      @connection-deleted="delete_connection"
      @connection-delete-many="delete_connection_many"></connection-list>
    <connection-list
      title="Failed Connections:"
      editable="false"
      :list="failed_connections"
      @connection-editted="update_connection"
      @connection-deleted="delete_connection"
      @connection-delete-many="delete_connection_many"></connection-list>

    <p>Add connection from invitation:</p>
    <el-form @submit.native.prevent>
      <el-form-item
        label="Invitation URL:">
        <el-input
          style="width: 300px;"
          v-model="invitation">
          <el-button
            slot="append"
            type="primary"
            icon="el-icon-plus"
            @click="recieve_invitation">Add</el-button>
        </el-input>
      </el-form-item>
    </el-form>
  </el-row>
</template>

<script>
import ConnectionList from './ConnectionList.vue';
import share from '@/share.js';
import message_bus from '@/message_bus.js';

export const protocol = 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-connections/0.1';
export const metadata = {
  menu: {
    label: 'Connections',
    icon: 'el-icon-user',
    group: 'Agent to Agent',
    priority: 30,
    required_protocols: [
      protocol
    ]
  },
  types: {
    connection: protocol + '/connection',
    get_list: protocol + '/connection-get-list',
    connection_list: protocol + '/connection-list',
    update: protocol + '/update',
    delete: protocol + '/delete',
    delete_many: protocol + '/delete-many',
    receive_invitation: protocol + '/receive_invitation',
    ack: protocol + '/ack',
  }
};

export const shared = {
  data: {
    connections: []
  },
  computed: {
    active_connections: function() {
        return Object.values(this.connections).filter(
            conn => "state" in conn && conn.state === "active"
        );
    },
    id_to_connection: function(connection_id) {
      let map = {};
      this.connections.forEach((connection) => {
        map[connection.connection_id] = connection;
      })
      console.log(map);
      return map;
    }
  },
  listeners: {
    [metadata.types.connection_list]: (share, msg) => share.connections = msg.results,
    [metadata.types.connection]: (share, msg) => share.fetch_connections(),
    [metadata.types.ack]: (share, msg) => share.fetch_connections(),
  },
  methods: {
    fetch_connections: ({send}) => {
      send({
        "@type": metadata.types.get_list
      });
    }
  }
};

export default {
  name: 'connections',
  components: {
    ConnectionList
  },
  mixins: [
    message_bus(),
    share({
      use: ['connections', 'active_connections'],
      actions: ['fetch_connections']
    })
  ],
  data: function() {
    return {
      invitation: '',
    }
  },
  created: async function() {
    await this.ready();
    this.fetch_connections();
  },
  computed: {
    pending_connections: function() {
      return Object.values(this.connections).filter(
        conn => "state" in conn &&
        conn.state !== "active" &&
        conn.state !== "invitation" &&
        conn.state !== "error"
      );
    },
    failed_connections: function() {
      return Object.values(this.connections).filter(
        conn => "state" in conn && conn.state === "error"
      );
    }
  },
  methods: {
    update_connection: function(form) {
      let query_msg = {
        "@type": metadata.types.update,
        "connection_id": form.connection_id,
        "label": form.label,
        "role": form.role,
      };
      this.send_message(query_msg);
    },
    delete_connection: function(connection) {
      let query_msg = {
        "@type": metadata.types.delete,
        "connection_id": connection.connection_id,
      };
      this.send_message(query_msg);
    },
    delete_connection_many: function(connection_ids) {
      let query_msg = {
        "@type": metadata.types.delete_many,
        "connection_ids": connection_ids,
      };
      this.send_message(query_msg);
    },
    recieve_invitation: function() {
      let receive_invite_msg = {
        "@type": metadata.types.receive_invitation,
        "invitation": this.invitation,
        "accept": "auto"
      };
      this.send_message(receive_invite_msg);
      this.invitation = "";
      setTimeout(() => {
        return this.fetch_connections();
      }, 4000);
    },
  },
}
</script>
