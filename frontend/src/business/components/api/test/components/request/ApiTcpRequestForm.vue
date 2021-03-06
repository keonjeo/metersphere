<template>
  <el-form class="tcp" :model="request" :rules="rules" ref="request" label-width="auto" :disabled="isReadOnly">

    <el-form-item :label="$t('api_test.request.name')" prop="name">
      <el-input v-model="request.name" maxlength="300" show-word-limit/>
    </el-form-item>

    <el-form-item label="TCPClient" prop="classname">
      <el-select v-model="request.classname" style="width: 100%">
        <el-option v-for="c in classes" :key="c" :label="c" :value="c"/>
      </el-select>
    </el-form-item>

    <el-row :gutter="10">
      <el-col :span="16">
        <el-form-item :label="$t('api_test.request.tcp.server')" prop="server">
          <el-input v-model="request.server" maxlength="300" show-word-limit/>
        </el-form-item>
      </el-col>
      <el-col :span="8">
        <el-form-item :label="$t('api_test.request.tcp.port')" prop="port" label-width="60px">
          <el-input-number v-model="request.port" controls-position="right" :min="0" :max="65535"/>
        </el-form-item>
      </el-col>
    </el-row>

    <el-row :gutter="10">
      <el-col :span="6">
        <el-form-item :label="$t('api_test.request.tcp.connect')" prop="ctimeout">
          <el-input-number v-model="request.ctimeout" controls-position="right" :min="0"/>
        </el-form-item>
      </el-col>
      <el-col :span="6">
        <el-form-item :label="$t('api_test.request.tcp.response')" prop="timeout">
          <el-input-number v-model="request.timeout" controls-position="right" :min="0"/>
        </el-form-item>
      </el-col>
      <el-col :span="6">
        <el-form-item :label="$t('api_test.request.tcp.so_linger')" prop="soLinger">
          <el-input v-model="request.soLinger"/>
        </el-form-item>
      </el-col>
      <el-col :span="6">
        <el-form-item :label="$t('api_test.request.tcp.eol_byte')" prop="eolByte">
          <el-input v-model="request.eolByte"/>
        </el-form-item>
      </el-col>
    </el-row>

    <el-row :gutter="10">
      <el-col :span="6">
        <el-form-item>
          <el-switch
            v-model="request.useEnvironment"
            :active-text="$t('api_test.request.refer_to_environment')"
            @change="useEnvironmentChange">
          </el-switch>
        </el-form-item>
      </el-col>
      <el-col :span="6">
        <el-form-item>
          <el-switch
            v-model="request.reUseConnection"
            :active-text="$t('api_test.request.tcp.re_use_connection')">
          </el-switch>
        </el-form-item>
      </el-col>
      <el-col :span="6">
        <el-form-item>
          <el-switch
            v-model="request.closeConnection"
            :active-text="$t('api_test.request.tcp.close_connection')">
          </el-switch>
        </el-form-item>
      </el-col>
      <el-col :span="6">
        <el-form-item>
          <el-switch
            v-model="request.nodelay"
            :active-text="$t('api_test.request.tcp.no_delay')">
          </el-switch>
        </el-form-item>
      </el-col>
    </el-row>

    <el-form-item :label="$t('api_test.request.tcp.request')" prop="request">
      <el-input type="textarea" v-model="request.request" :autosize="{minRows: 4, maxRows: 6}">
      </el-input>
    </el-form-item>

    <el-row :gutter="10">
      <el-col :span="12">
        <el-form-item :label="$t('api_test.request.tcp.username')" prop="username">
          <el-input v-model="request.username" maxlength="100" show-word-limit/>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item :label="$t('api_test.request.tcp.password')" prop="password">
          <el-input v-model="request.password" maxlength="30" show-word-limit show-password
                    autocomplete="new-password"/>
        </el-form-item>
      </el-col>
    </el-row>

    <el-button :disabled="!request.enable || !scenario.enable || isReadOnly" class="debug-button" size="small"
               type="primary" @click="runDebug">{{ $t('api_test.request.debug') }}
    </el-button>

    <el-tabs v-model="activeName">
      <el-tab-pane :label="$t('api_test.request.assertions.label')" name="assertions">
        <ms-api-assertions :is-read-only="isReadOnly" :assertions="request.assertions"/>
      </el-tab-pane>
      <el-tab-pane :label="$t('api_test.request.extract.label')" name="extract">
        <ms-api-extract :is-read-only="isReadOnly" :extract="request.extract"/>
      </el-tab-pane>
      <el-tab-pane :label="$t('api_test.request.processor.pre_exec_script')" name="beanShellPreProcessor">
        <ms-jsr233-processor :is-read-only="isReadOnly" :jsr223-processor="request.jsr223PreProcessor"/>
      </el-tab-pane>
      <el-tab-pane :label="$t('api_test.request.processor.post_exec_script')" name="beanShellPostProcessor">
        <ms-jsr233-processor :is-read-only="isReadOnly" :jsr223-processor="request.jsr223PostProcessor"/>
      </el-tab-pane>
    </el-tabs>
  </el-form>
</template>

<script>
import {Scenario, TCPRequest} from "@/business/components/api/test/model/ScenarioModel";
import MsApiAssertions from "@/business/components/api/test/components/assertion/ApiAssertions";
import MsApiExtract from "@/business/components/api/test/components/extract/ApiExtract";
import MsJsr233Processor from "@/business/components/api/test/components/processor/Jsr233Processor";

export default {
  name: "MsApiTcpRequestForm",
  components: {MsJsr233Processor, MsApiExtract, MsApiAssertions},
  props: {
    request: TCPRequest,
    scenario: Scenario,
    isReadOnly: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      activeName: "assertions",
      classes: TCPRequest.CLASSES,
      rules: {
        server: [
          {
            required: true,
            message: this.$t('commons.required', [this.$t('api_test.request.tcp.server')]),
            trigger: 'blur'
          }
        ],
      }
    }
  },

  methods: {
    useEnvironmentChange(value) {
      if (value && !this.request.environment) {
        this.$error(this.$t('api_test.request.please_add_environment_to_scenario'), 2000);
        this.request.useEnvironment = false;
      }
      this.$refs["request"].clearValidate();
    },
    runDebug() {
      this.$emit('runDebug');
    }
  },
}
</script>

<style scoped>
.tcp >>> .el-input-number {
  width: 100%;
}
</style>
