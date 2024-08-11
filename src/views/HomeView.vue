<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useClipboard } from '@vueuse/core'
import { ElMessage } from 'element-plus'

const defaultBackend = import.meta.env.VITE_SUBCONVERTER_DEFAULT_BACKEND + '/sub?'

const options = {
  clientTypes: {
    Clash: 'clash',
    Surge: 'surge&ver=4',
    Quantumult: 'quan',
    QuantumultX: 'quanx',
    Mellow: 'mellow',
    Surfboard: 'surfboard',
    Loon: 'loon',
    singbox: 'singbox',
    ss: 'ss',
    ssd: 'ssd',
    sssub: 'sssub',
    ssr: 'ssr',
    ClashR: 'clashr',
    V2Ray: 'v2ray',
    Trojan: 'trojan',
    Surge3: 'surge&ver=3'
  },
  backendOptions: [{ value: 'http://127.0.0.1:25500/sub?' }],
  remoteConfig: [
    {
      label: 'universal',
      options: [
        {
          label: 'No-Urltest',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/universal/no-urltest.ini'
        },
        {
          label: 'Urltest',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/universal/urltest.ini'
        }
      ]
    },
    {
      label: 'customized',
      options: [
        {
          label: 'Maying',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/maying.ini'
        },
        {
          label: 'Ytoo',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ytoo.ini'
        },
        {
          label: 'FlowerCloud',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/flowercloud.ini'
        },
        {
          label: 'Nexitally',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/nexitally.ini'
        },
        {
          label: 'SoCloud',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/socloud.ini'
        },
        {
          label: 'ARK',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ark.ini'
        },
        {
          label: 'ssrCloud',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ssrcloud.ini'
        }
      ]
    },
    {
      label: 'Special',
      options: [
        {
          label: 'NeteaseUnblock(仅规则，No-Urltest)',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/special/netease.ini'
        },
        {
          label: 'Basic(仅GEOIP CN + Final)',
          value:
            'https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/special/basic.ini'
        }
      ]
    }
  ]
}

const form = reactive({
  advanced: 1,
  sourceSubUrl: '',
  clientType: 'clash',
  customBackend: '',
  remoteConfig: '',
  includeRemarks: '',
  excludeRemarks: '',
  filename: '',
  nodeList: false,
  extraset: false,
  emoji: false,
  scv: false,
  udp: false,
  appendType: false,
  sort: false,
  fdn: false,
  expand: false
})

const dialogUploadConfigVisible = ref(false)
const backendVersion = ref('0.9.0')

const customSubUrl = ref('')
const curtomShortSubUrl = ref('')
const loading = ref(false)
const { copy, copied } = useClipboard({ source: customSubUrl })

const cancelUploadConfig = () => {
  uploadConfig = ''
  dialogUploadConfigVisible.value = false
}

const saveSubUrl = () => {
  if (this.form.sourceSubUrl !== '') {
    this.setLocalStorageItem('sourceSubUrl', this.form.sourceSubUrl)
  }
}

const makeUrl = () => {
  if (form.sourceSubUrl === '' || form.clientType === '') {
    ElMessage.error('订阅链接与客户端为必填项')
    return false
  }
  const backend = form.customBackend || defaultBackend
  const sourceSub = form.sourceSubUrl.replace(/(\n|\r|\n\r)/g, '|')

  customSubUrl.value =
    backend +
    'target=' +
    form.clientType +
    '&url=' +
    encodeURIComponent(sourceSub) +
    '&insert=' +
    form.insert

  if (form.advanced === 2) {
    if (this.form.remoteConfig) {
      this.customSubUrl += '&config=' + encodeURIComponent(this.form.remoteConfig)
    }
    if (this.form.excludeRemarks) {
      this.customSubUrl += '&exclude=' + encodeURIComponent(this.form.excludeRemarks)
    }
    if (this.form.includeRemarks) {
      this.customSubUrl += '&include=' + encodeURIComponent(this.form.includeRemarks)
    }
    if (this.form.filename) {
      this.customSubUrl += '&filename=' + encodeURIComponent(this.form.filename)
    }
    if (this.form.appendType) {
      this.customSubUrl += '&append_type=' + this.form.appendType.toString()
    }

    this.customSubUrl +=
      '&emoji=' +
      this.form.emoji.toString() +
      '&list=' +
      this.form.nodeList.toString() +
      '&tfo=' +
      this.form.tfo.toString() +
      '&scv=' +
      this.form.scv.toString() +
      '&fdn=' +
      this.form.fdn.toString() +
      '&expand=' +
      this.form.expand.toString() +
      '&sort=' +
      this.form.sort.toString()

    if (this.needUdp) {
      this.customSubUrl += '&udp=' + this.form.udp.toString()
    }

    if (this.form.tpl.surge.doh === true) {
      this.customSubUrl += '&surge.doh=true'
    }

    if (this.form.clientType === 'clash') {
      if (this.form.tpl.clash.doh === true) {
        this.customSubUrl += '&clash.doh=true'
      }

      this.customSubUrl += '&new_name=' + this.form.new_name.toString()
    }

    this.customParams
      .filter((param) => param.name && param.value)
      .forEach((param) => {
        this.customSubUrl += `&${encodeURIComponent(param.name)}=${encodeURIComponent(param.value)}`
      })
  }

  copy()
  if (copied) {
    ElMessage.success('定制订阅已复制到剪贴板')
  }
}
const makeShortUrl = () => {
  if (this.customSubUrl === '') {
    this.$message.warning('请先生成订阅链接，再获取对应短链接')
    return false
  }

  this.loading = true

  let data = new FormData()
  data.append('longUrl', btoa(this.customSubUrl))

  this.$axios
    .post(shortUrlBackend, data, {
      header: {
        'Content-Type': 'application/form-data; charset=utf-8'
      }
    })
    .then((res) => {
      if (res.data.Code === 1 && res.data.ShortUrl !== '') {
        this.curtomShortSubUrl = res.data.ShortUrl
        this.$copyText(res.data.ShortUrl)
        this.$message.success('短链接已复制到剪贴板')
      } else {
        this.$message.error('短链接获取失败：' + res.data.Message)
      }
    })
    .catch(() => {
      this.$message.error('短链接获取失败')
    })
    .finally(() => {
      this.loading = false
    })
}

const clashInstall = () => {
  if (this.customSubUrl === '') {
    this.$message.error('请先填写必填项，生成订阅链接')
    return false
  }

  const url = 'clash://install-config?url='
  window.open(
    url +
      encodeURIComponent(this.curtomShortSubUrl !== '' ? this.curtomShortSubUrl : this.customSubUrl)
  )
}
</script>

<template>
  <el-row style="margin-top: 10px">
    <el-col>
      <el-card>
        <template v-slot:header>
          <div>
            Subscription Converter
            <!-- <svg-icon icon-class="github" style="margin-left: 20px" @click="goToProject" /> -->
            <div style="display: inline-block; position: absolute; right: 20px">
              {{ backendVersion }}
            </div>
          </div>
        </template>
        <el-container>
          <el-form :model="form" label-width="140px" label-position="left" style="width: 100%">
            <el-form-item label="模式设置:" prop="advanced">
              <el-radio v-model="form.advanced" :label="1">基础模式</el-radio>
              <el-radio v-model="form.advanced" :label="2">进阶模式</el-radio>
            </el-form-item>
            <el-form-item label="订阅链接:" prop="sourceSubUrl">
              <el-input
                v-model="form.sourceSubUrl"
                type="textarea"
                :rows="3"
                placeholder="支持订阅或ss/ssr/vmess链接，多个链接每行一个或用 | 分隔"
                @blur="saveSubUrl"
              />
            </el-form-item>
            <el-form-item label="客户端:">
              <el-select v-model="form.clientType" style="width: 100%">
                <el-option
                  v-for="(v, k) in options.clientTypes"
                  :key="k"
                  :label="k"
                  :value="v"
                ></el-option>
              </el-select>
            </el-form-item>

            <!-- <div v-if="form.advanced === 2">
              <el-form-item label="后端地址:">
                <el-autocomplete
                  style="width: 100%"
                  v-model="form.customBackend"
                  :fetch-suggestions="backendSearch"
                  placeholder="动动小手，（建议）自行搭建后端服务。例：http://127.0.0.1:25500/sub?"
                >
                  <template v-slot:append>
                    <el-button @click="gotoGayhub" icon="el-icon-link">前往项目仓库</el-button>
                  </template>
                </el-autocomplete>
              </el-form-item>
              <el-form-item label="远程配置:">
                <el-select
                  v-model="form.remoteConfig"
                  allow-create
                  filterable
                  placeholder="请选择"
                  style="width: 100%"
                >
                  <el-option-group
                    v-for="group in options.remoteConfig"
                    :key="group.label"
                    :label="group.label"
                  >
                    <el-option
                      v-for="item in group.options"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-option-group>
                  <template v-slot:append>
                    <el-button @click="gotoRemoteConfig" icon="el-icon-link">配置示例</el-button>
                  </template>
                </el-select>
              </el-form-item>
              <el-form-item label="Include:">
                <el-input
                  v-model="form.includeRemarks"
                  placeholder="节点名包含的关键字，支持正则"
                />
              </el-form-item>
              <el-form-item label="Exclude:">
                <el-input
                  v-model="form.excludeRemarks"
                  placeholder="节点名不包含的关键字，支持正则"
                />
              </el-form-item>
              <el-form-item label="FileName:">
                <el-input v-model="form.filename" placeholder="返回的订阅文件名" />
              </el-form-item>

              <el-form-item v-for="(param, i) in customParams" :key="i">
                <template v-slot:label>
                  <el-input v-model="param.name" placeholder="自定义参数名">
                    <template v-slot:suffix>
                      <div style="width: 10px">:</div>
                    </template>
                  </el-input>
                </template>
                <el-input v-model="param.value" placeholder="自定义参数内容">
                  <template v-slot:suffix>
                    <el-button
                      type="text"
                      icon="el-icon-delete"
                      style="margin-right: 5px"
                      @click="customParams.splice(i, 1)"
                    />
                  </template>
                </el-input>
              </el-form-item>

              <el-form-item label-width="0px">
                <el-row type="flex">
                  <el-col>
                    <el-checkbox
                      v-model="form.nodeList"
                      label="输出为 Node List"
                      border
                    ></el-checkbox>
                  </el-col>
                  <el-popover placement="bottom" v-model="form.extraset">
                    <el-row>
                      <el-checkbox v-model="form.emoji" label="Emoji"></el-checkbox>
                    </el-row>
                    <el-row>
                      <el-checkbox v-model="form.scv" label="跳过证书验证"></el-checkbox>
                    </el-row>
                    <el-row>
                      <el-checkbox
                        v-model="form.udp"
                        @change="needUdp = true"
                        label="启用 UDP"
                      ></el-checkbox>
                    </el-row>
                    <el-row>
                      <el-checkbox v-model="form.appendType" label="节点类型"></el-checkbox>
                    </el-row>
                    <el-row>
                      <el-checkbox v-model="form.sort" label="排序节点"></el-checkbox>
                    </el-row>
                    <el-row>
                      <el-checkbox v-model="form.fdn" label="过滤非法节点"></el-checkbox>
                    </el-row>
                    <el-row>
                      <el-checkbox v-model="form.expand" label="规则展开"></el-checkbox>
                    </el-row>
                    <template v-slot:reference>
                      <el-button>更多选项</el-button>
                    </template>
                  </el-popover>
                  <el-popover placement="bottom" style="margin-left: 10px">
                    <el-row>
                      <el-checkbox v-model="form.tpl.surge.doh" label="Surge.DoH"></el-checkbox>
                    </el-row>
                    <el-row>
                      <el-checkbox v-model="form.tpl.clash.doh" label="Clash.DoH"></el-checkbox>
                    </el-row>
                    <el-row>
                      <el-checkbox v-model="form.insert" label="网易云"></el-checkbox>
                    </el-row>
                    <template v-slot:reference>
                      <el-button>定制功能</el-button>
                    </template>
                  </el-popover>
                  <el-popover placement="top-end" title="添加自定义转换参数" trigger="hover">
                    <el-link
                      type="primary"
                      :href="subDocAdvanced"
                      target="_blank"
                      icon="el-icon-info"
                      >参考文档</el-link
                    >
                    <template v-slot:reference>
                      <el-button @click="addCustomParam" style="margin-left: 10px">
                        <i class="el-icon-plus"></i>
                      </el-button>
                    </template>
                  </el-popover>
                </el-row>
              </el-form-item>
            </div> -->

            <div style="margin-top: 50px"></div>

            <el-divider content-position="center">
              <i class="el-icon-magic-stick"></i>
            </el-divider>

            <el-form-item label="定制订阅:">
              <el-input class="copy-content" disabled v-model="customSubUrl">
                <template v-slot:append>
                  <!-- <el-button
                    v-clipboard:copy="customSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                    >复制</el-button
                  > -->
                </template>
              </el-input>
            </el-form-item>
            <el-form-item label="订阅短链:">
              <el-input class="copy-content" disabled v-model="curtomShortSubUrl">
                <template v-slot:append>
                  <!-- <el-button
                    v-clipboard:copy="curtomShortSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                    >复制</el-button
                  > -->
                </template>
              </el-input>
            </el-form-item>

            <el-form-item label-width="0px" style="margin-top: 40px; text-align: center">
              <el-button
                style="width: 140px"
                type="danger"
                @click="makeUrl"
                :disabled="form.sourceSubUrl.length === 0"
                >生成订阅链接</el-button
              >
              <el-button
                style="width: 140px"
                type="danger"
                @click="makeShortUrl"
                :loading="loading"
                :disabled="customSubUrl.length === 0"
                >生成短链接</el-button
              >
              <!-- <el-button style="width: 140px" type="primary" @click="surgeInstall" icon="el-icon-connection">一键导入Surge</el-button> -->
            </el-form-item>

            <el-form-item label-width="0px" style="text-align: center">
              <el-button
                style="width: 140px"
                type="primary"
                @click="dialogUploadConfigVisible = true"
                icon="el-icon-upload"
                :loading="loading"
                >上传配置</el-button
              >
              <el-button
                style="width: 140px"
                type="primary"
                @click="clashInstall"
                icon="el-icon-connection"
                :disabled="customSubUrl.length === 0"
                >一键导入 Clash</el-button
              >
            </el-form-item>
            <el-form-item label-width="0px" style="text-align: center">
              <el-button
                style="width: 290px"
                type="primary"
                @click="dialogLoadConfigVisible = true"
                icon="el-icon-copy-document"
                :loading="loading"
                >从 URL 解析</el-button
              >
            </el-form-item>
          </el-form>
        </el-container>
      </el-card>
    </el-col>
  </el-row>

  <!-- <el-dialog
    v-model:visible="dialogUploadConfigVisible"
    :show-close="false"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    width="700px"
  >
    <template v-slot:title>
      <div>
        Remote config upload
        <el-popover trigger="hover" placement="right" style="margin-left: 10px">
          <el-link type="primary" :href="sampleConfig" target="_blank" icon="el-icon-info"
            >参考配置</el-link
          >
          <template v-slot:reference>
            <i class="el-icon-question"></i>
          </template>
        </el-popover>
      </div>
    </template>
    <el-form label-position="left">
      <el-form-item prop="uploadConfig">
        <el-input
          v-model="uploadConfig"
          type="textarea"
          :autosize="{ minRows: 15, maxRows: 30 }"
          maxlength="10000"
          show-word-limit
        ></el-input>
      </el-form-item>
    </el-form>
    <template v-slot:footer>
      <div class="dialog-footer">
        <el-button
          @click="
            uploadConfig = ''
            dialogUploadConfigVisible = false
          "
          >取 消</el-button
        >
        <el-button type="primary" @click="confirmUploadConfig" :disabled="uploadConfig.length === 0"
          >确 定</el-button
        >
      </div>
    </template>
  </el-dialog>

  <el-dialog
    v-model:visible="dialogLoadConfigVisible"
    :show-close="false"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    width="700px"
  >
    <template v-slot:title>
      <div>解析 Subconverter 链接</div>
    </template>
    <el-form label-position="left" :inline="true">
      <el-form-item prop="uploadConfig" label="订阅链接：" label-width="85px">
        <el-input v-model="loadConfig" style="width: 565px"></el-input>
      </el-form-item>
    </el-form>
    <template v-slot:footer>
      <div class="dialog-footer">
        <el-button
          @click="
            loadConfig = ''
            dialogLoadConfigVisible = false
          "
          >取 消</el-button
        >
        <el-button type="primary" @click="confirmLoadConfig" :disabled="loadConfig.length === 0"
          >确 定</el-button
        >
      </div>
    </template>
  </el-dialog> -->
</template>
