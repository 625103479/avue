<template>
  <el-collapse-transition>
    <div :class="b()"
         v-show="searchShow && searchFlag">
      <avue-form :option="option"
                 ref="form"
                 @submit="searchChange"
                 @reset-change="resetChange"
                 v-model="searchForm">
        <template slot="menuForm"
                  slot-scope="{size}">
          <slot name="searchMenu"
                :row="searchForm"
                :size="size"></slot>
        </template>
        <template :slot="item.prop"
                  slot-scope="scope"
                  v-for="item in columnOption">
          <slot :value="scope.value"
                :column="scope.column"
                :dic="scope.dic"
                :size="scope.size"
                :label="scope.label"
                :disabled="scope.disabled"
                :row="searchForm"
                :name="item.prop"
                v-if="item.searchslot"></slot>
        </template>
        <template slot="search"
                  slot-scope="{}">
          <slot name="search"
                :row="searchForm"
                :size="crud.controlSize"></slot>
        </template>
      </avue-form>
    </div>
  </el-collapse-transition>
</template>

<script>
import cteate from "core/create";
import { vaildData } from "utils/util";
import { validatenull } from "utils/validate";
import locale from "../../core/common/locale";
import {
  formInitVal,
  getSearchType,
  getType,
  getPlaceholder
} from "core/dataformat";
import config from "./config";
export default cteate({
  name: "crud__search",
  inject: ["crud"],
  mixins: [locale],
  data () {
    return {
      config: config,
      defaultForm: {
        searchForm: {}
      },
      searchShow: true,
      searchForm: {}
    };
  },
  props: {
    value: {
      type: Object,
      default: () => {
        return {};
      }
    }
  },
  watch: {
    'crud.propOption': {
      handler () {
        this.dataformat();
      },
      immediate: true
    },
    searchForm: {
      handler () {
        this.$emit("input", this.searchForm);
      },
      deep: true
    }
  },
  created () {
    this.init();
  },
  computed: {
    columnOption () {
      return this.option.column || []
    },
    option () {
      const option = this.crud.option;
      const detailColumn = (list = []) => {
        let column = [];
        list.forEach(ele => {
          if (ele.search) {
            ele = Object.assign(ele, {
              type: getSearchType(ele),
              multiple: ele.searchMultiple,
              detail: false,
              span: ele.searchSpan || this.config.searchSpan,
              labelWidth: ele.searchLabelWidth || option.searchLabelWidth || this.config.searchLabelWidth,
              tip: ele.searchTip,
              placeholder: getPlaceholder(ele, 'search'),
              filterable: ele.searchFilterable,
              tipPlacement: ele.searchTipPlacement,
              filterMethod: ele.searchFilterMethod,
              checkStrictly: ele.searchCheckStrictly || option.searchCheckStrictly,
              gutter: ele.searchGutter || option.searchGutter,
              tags: ele.searchTags,
              row: ele.searchRow,
              size: ele.searchSize,
              formslot: ele.searchslot,
              rules: ele.searchRules,
              disabled: ele.searchDisabled,
              readonly: ele.searchReadonly,
              value: ele.searchValue
            })
            let whiteList = ['display', 'disabled', 'readonly']
            whiteList.forEach(key => {
              delete ele[key]
            })
            column.push(ele);
          }
        })
        return column;
      }
      const dataDetail = (list) => {
        let result = this.deepClone(list);
        result.translate = false;
        if (result.group) {
          delete result.group;
        }
        result.column = detailColumn(this.deepClone(this.crud.columnFormOption))
        result = Object.assign(result, {
          tabs: false,
          printBtn: false,
          mockBtn: false,
          size: option.searchSize || this.crud.isMediumSize,
          gutter: option.searchGutter || this.config.searchGutter,
          labelWidth: option.searchLabelWidth || this.config.searchLabelWidth,
          submitText: this.vaildData(option.searchBtnText, this.t('crud.searchBtn')),
          submitBtn: this.vaildData(option.searchBtn, this.config.searchSubBtn),
          submitIcon: option.searchBtnIcon || this.config.searchBtnIcon,
          emptyText: this.vaildData(option.emptyBtnText, this.t('crud.emptyBtn')),
          emptyBtn: this.vaildData(option.emptyBtn, this.config.emptyBtn),
          emptyIcon: option.emptyBtnIcon || this.config.emptyBtnIcon,
          menuSpan: option.searchMenuSpan,
          dicFlag: false,
          dicData: this.crud.DIC
        })
        return result;
      }
      return dataDetail(option)
    },
    searchSlot () {
      return !validatenull(this.$slots.search);
    },
    searchFlag () {
      if (this.searchSlot) return true;
      else return !validatenull(this.searchForm);
    }
  },
  methods: {
    //初始化
    init () {
      //扩展搜索的相关api
      this.crud.searchChange = this.searchChange;
      this.crud.searchReset = this.searchReset;
    },
    // 搜索回调
    searchChange (form, done) {
      this.crud.$emit("search-change", form, done);
    },
    // 搜索清空
    resetChange () {
      this.crud.$emit("search-reset", this.defaultForm.tableForm);
    },
    // 搜索清空
    searchReset () {
      this.$refs.form.resetFields();
    },
    handleSearchShow () {
      this.searchShow = !this.searchShow;
    },
    dataformat () {
      this.defaultForm = formInitVal(this.option.column);
      this.searchForm = this.deepClone(this.defaultForm.tableForm);
      this.searchShow = vaildData(
        this.crud.tableOption.searchShow,
        this.crud.config.searchShow
      );
    }
  }
});
</script>
