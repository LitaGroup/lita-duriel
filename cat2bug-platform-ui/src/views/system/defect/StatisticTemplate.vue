<template>
  <div class="app-container">
    <el-row class="project-add-page-header">
      <el-page-header @back="goBack" :content="$t('defect.statistic-template')">
      </el-page-header>
    </el-row>
    <h4 class="first-title">{{ $t('defect.statistic.view') }}</h4>
    <div class="view-panel">
      <cat2-bug-statistic :read="true" v-model="selectTemplate" :draggable="true" />
    </div>
    <h4>{{ $t('defect.statistic.select') }}</h4>
    <div class="templates-panel">
      <cat2-bug-statistic show-type="all" :read="true" :statistic-tools="[]" @click-template-node="clickAddTemplate" />
    </div>
  </div>
</template>

<script>
import ProjectLabel from "@/components/Project/ProjectLabel";
import Cat2BugStatistic from "@/components/Cat2BugStatistic"
import {addStatistic, listStatistic} from "@/api/system/statistic/template";

// 统计模版
export default {
  name: "StatisticTemplate",
  components: {ProjectLabel,Cat2BugStatistic},
  data() {
    return {
      selectTemplate: [],
    }
  },
  computed: {
    /** 获取用户id */
    userId: function() {
      return this.$store.state.user.id;
    },
    /** 获取项目id */
    projectId: function() {
      return parseInt(this.$store.state.user.config.currentProjectId);
    },
  },
  created() {
    this.getTemplate();
  },
  methods: {
    getTemplate() {
      let params = {
        userId: this.userId,
        projectId: this.projectId,
        moduleType: 1
      }
      listStatistic(params).then(res=>{
        if(res.rows.length>0) {
          this.selectTemplate = JSON.parse(res.rows[0].statisticTemplatConfig);
        } else {
          this.selectTemplate = [];
        }
      })
    },
    clickAddTemplate: function (e,template){
      if(this.selectTemplate.filter(t=>t.name==template.name).length==0) {
        this.selectTemplate.push(template);
        let params = {
          userId: this.userId,
          projectId: this.projectId,
          moduleType: 1,
          statisticTemplatConfig: JSON.stringify(this.selectTemplate)
        }
        addStatistic(params).then(res => {
          this.$message.success(this.$i18n.t('defect.statistic-save-success').toString());
        });
      } else {
        this.$message.warning(this.$i18n.t('defect.statistic-exists-warning').toString());
      }
      e.stopPropagation();
    },
    /** 返回 */
    goBack() {
      this.$router.back();
    },
  }
}
</script>

<style lang="scss" scoped>
  .first-title {
    margin-top: 10px;
  }
  .app-container {
    width: 100%;
    display: flex;
    flex-direction: column;
    position: absolute;
    top: 0px;
    bottom: 0px;
  }
  .view-panel, .templates-panel {
    width: 100%;
    min-height: 162px;
    background-color: #F2F6FC;
    border-radius: 5px;
    padding: 10px 15px;
    ::v-deep .statistic-box:hover {
      cursor: pointer;
    }
  }
  .templates-panel {
    flex: 1;
    overflow-y: scroll;
  }
</style>
