<template>
  <el-dialog :title="title" width="500px" :visible.sync="visible">
    <el-form :model="table" :rules="rules" label-width="90px" size="mini" ref="form">
      <el-form-item label="显示标题">
        <el-input v-model="table.label"></el-input>
      </el-form-item>
      <el-form-item label="实体对象名">
        <el-input v-model="table.name"></el-input>
      </el-form-item>
      <el-form-item label="数据库表名">
        <el-input v-model="table.code"></el-input>
      </el-form-item>
      <el-row>
        <template v-for="tag in tableTags">
          <el-col :span="12" :key="tag.name">
            <el-form-item :label="tag.name" :class="{ ['extra-input']: tag.type != 'NONE' && tagStatus[tag.name] }">
              <el-switch v-model="tagStatus[tag.name]" @input="e => setTagStatus(tag, e)" />
              <template v-if="tag.type != 'NONE'">
                <el-input v-if="tagStatus[tag.name]" v-model="table.tags[tag.name]" />
              </template>
            </el-form-item>
          </el-col>
        </template>
      </el-row>
      <el-form-item label="描述">
        <el-input type="textarea" :rows="2" placeholder="请输入内容" v-model="table.comment" />
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取 消</el-button>
      <el-button type="primary" @click="save">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { mapState, mapGetters, mapMutations } from "vuex";
import { createTable } from "../utils";

export default {
  name: "TableDialog",
  data() {
    return {
      visible: false,
      title: "",
      unit: null,
      table: { tags: [] },
      oldTable: null,
      tagStatus: {},
      rules: {
        name: [{ required: true, message: "请输入名称", trigger: "blur" }],
        label: [{ required: true, message: "请输入显示标题", trigger: "blur" }],
        code: [{ required: true, message: "请输入数据库表名", trigger: "blur" }]
      }
    };
  },
  computed: {
    ...mapState(["world"]),
    ...mapGetters(["model", "tableTags"])
  },
  methods: {
    ...mapMutations(["setChanged"]),
    newTable(unit) {
      this.unit = unit;
      this.title = "新建数据表";
      this.oldTable = null;
      this.table = createTable(this.model);
      let tagStatus = {};
      if (this.tableTags) {
        this.tableTags.forEach(tag => {
          tagStatus[tag.name] = false;
        });
      }
      this.tagStatus = tagStatus;
      this.visible = true;
    },
    editTable(table) {
      if (table.tags == null) table.tags = {};
      this.title = "编辑数据表";
      this.oldTable = table;
      this.table = this._.cloneDeep(table);
      let tagStatus = {};
      if (this.tableTags) {
        this.tableTags.forEach(tag => {
          tagStatus[tag.name] = this._.has(table.tags, tag.name);
        });
      }
      this.tagStatus = tagStatus;
      this.visible = true;
    },
    save() {
      this.$refs.form.validate(valid => {
        if (valid) this.doSave();
      });
    },
    doSave() {
      if (this.oldTable == null) {
        this.unit.children.unshift(this.table);
        this.world.tables[this.table.id] = this.table;
      } else {
        this._.assign(this.oldTable, this.table);
      }
      this.setChanged(true);
      this.table = { tags: [] };
      this.visible = false;
    },

    setTagStatus(tag, enabled) {
      if (enabled) {
        this.$set(this.table.tags, tag.name, tag.defaultParam || "");
      } else {
        this.table.tags = this._.omit(this.table.tags, [tag.name]);
      }
    }
  }
};
</script>

<style lang="scss">
.el-select {
  display: block !important;
}
.extra-input > div {
  display: flex;
  align-items: center;
  .el-switch {
    margin-right: 5px;
  }
}
</style>
