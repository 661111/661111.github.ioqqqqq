<template>
  <div>
    <a-form :form="form" style="padding-bottom: 48px;">
      <a-form-item :label="$t('platform')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
        <a-radio-group name="platform" v-model="form.platform">
          <a-radio value="github">Github Pages</a-radio>
          <a-radio value="netlify">Netlify</a-radio>
          <a-radio value="coding">Coding Pages</a-radio>
          <a-radio value="gitee">Gitee Pages</a-radio>
          <a-radio value="sftp">SFTP</a-radio>
        </a-radio-group>
      </a-form-item>
      <a-form-item :label="$t('domain')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
        <a-input-group compact>
          <a-select v-model="protocol" style="width: 96px">
            <a-select-option value="https://">https://</a-select-option>
            <a-select-option value="http://">http://</a-select-option>
          </a-select>
          <a-input v-model="form.domain" placeholder="mydomain.com" style="width: calc(100% - 96px);" />
        </a-input-group>
      </a-form-item>
      <template v-if="['netlify'].includes(form.platform)">
        <a-form-item label="Site ID" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.netlifySiteId" />
        </a-form-item>
        <a-form-item label="Access Token" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false" v-if="remoteType === 'password'">
          <a-input v-model="form.netlifyAccessToken" :type="passVisible ? '' : 'password'">
            <a-icon class="icon" slot="addonAfter" :type="passVisible ? 'eye-invisible' : 'eye'" @click="passVisible = !passVisible" />
          </a-input>
        </a-form-item>
        <a-form-item label=" " :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a href="https://gridea.dev/netlify" target="_blank">如何配置？</a>
        </a-form-item>
      </template>
      <template v-if="['github', 'coding', 'gitee'].includes(form.platform)">
        <a-form-item :label="$t('repository')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.repository" />
        </a-form-item>
        <a-form-item :label="$t('branch')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.branch" placeholder="master" />
        </a-form-item>
        <a-form-item :label="$t('username')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.username" />
        </a-form-item>
        <a-form-item :label="$t('email')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.email" />
        </a-form-item>
        <a-form-item v-if="form.platform === 'coding'" :label="$t('tokenUsername')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.tokenUsername" />
        </a-form-item>
        <a-form-item :label="$t('token')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input :type="passVisible ? '' : 'password'" v-model="form.token">
            <a-icon class="icon" slot="addonAfter" :type="passVisible ? 'eye-invisible' : 'eye'" @click="passVisible = !passVisible" />
          </a-input>
        </a-form-item>
        <a-form-item label="CNAME" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.cname" placeholder="mydomain.com" />
        </a-form-item>
      </template>
      <template v-if="['sftp'].includes(form.platform)">
        <a-form-item label="Port" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input-number v-model="form.port" />
        </a-form-item>
        <a-form-item label="Server" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.server" />
        </a-form-item>
        <a-form-item label="Username" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.username" />
        </a-form-item>
        <a-form-item label="Connect Type" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-radio-group name="remoteType" v-model="remoteType">
            <a-radio value="password">Password</a-radio>
            <a-radio value="key">SSH Key</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item label="Password" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false" v-if="remoteType === 'password'">
          <a-input v-model="form.password" :type="passVisible ? '' : 'password'">
            <a-icon class="icon" slot="addonAfter" :type="passVisible ? 'eye-invisible' : 'eye'" @click="passVisible = !passVisible" />
          </a-input>
        </a-form-item>
        <a-form-item label="Private Key Path" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false" :help="$t('privateKeyTip')" v-else>
          <a-input v-model="form.privateKey" />
        </a-form-item>
        <a-form-item label="Remote Path" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false" :help="$t('remotePathTip')">
          <a-input v-model="form.remotePath" />
        </a-form-item>
      </template>
      <a-form-item v-if="form.platform !== 'sftp'" :label="$t('Proxy')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
        <a-radio-group name="Proxy" v-model="form.enabledProxy">
          <a-radio value="direct">Direct</a-radio>
          <a-radio value="proxy">Proxy</a-radio>
        </a-radio-group>
      </a-form-item>
      <template v-if="['proxy'].includes(form.enabledProxy) && form.platform !== 'sftp'">
        <a-form-item :label="$t('ProxyAddress')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input-group compact>
            <a-input v-model="form.proxyPath" />
          </a-input-group>
        </a-form-item>
        <a-form-item :label="$t('ProxyPort')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.proxyPort" />
        </a-form-item>
      </template>
      <footer-box>
        <div class="flex justify-between">
          <a-button :disabled="!canSubmit" :loading="detectLoading" @click="remoteDetect" style="margin-right: 16px;">{{ $t('testConnection') }}</a-button>
          <a-button :disabled="!canSubmit" @click="submit" type="primary">{{ $t('save') }}</a-button>
        </div>
      </footer-box>
    </a-form>
  </div>
</template>

<script lang="ts">
import { ipcRenderer, IpcRendererEvent } from 'electron'
import { Vue, Component, Watch } from 'vue-property-decorator'
import { State } from 'vuex-class'
import FooterBox from '../../../components/FooterBox/Index.vue'
import ga from '../../../helpers/analytics'
import { ISetting } from '../../../interfaces/setting'

@Component({
  components: {
    FooterBox,
  },
})
export default class BasicSetting extends Vue {
  @State('site') site!: any

  passVisible = false

  detectLoading = false

  formLayout = {
    label: { span: 6 },
    wrapper: { span: 12 },
  }

  protocol = 'https://'

  form: ISetting = {
    platform: 'github',
    domain: '',
    repository: '',
    branch: '',
    username: '',
    email: '',
    tokenUsername: '',
    token: '',
    cname: '',
    port: '22',
    server: '',
    password: '',
    privateKey: '',
    remotePath: '',
    proxyPath: '',
    proxyPort: '',
    enabledProxy: 'direct',
    netlifyAccessToken: '',
    netlifySiteId: '',
  }

  remoteType = 'password'

  get canSubmit() {
    const { form } = this
    const baseValid = form.domain
      && form.repository
      && form.branch
      && form.username
      && form.token
    const pagesPlatfomValid = baseValid && (form.platform === 'gitee' || form.platform === 'github' || (form.platform === 'coding' && form.tokenUsername))

    const sftpPlatformValid = ['sftp'].includes(form.platform)
      && form.port
      && form.server
      && form.username
      && form.remotePath
      && (form.password || form.privateKey)

    const netlifyPlatformValid = ['netlify'].includes(form.platform)
      && form.netlifyAccessToken
      && form.netlifySiteId

    return pagesPlatfomValid || sftpPlatformValid || netlifyPlatformValid
  }

  mounted() {
    const { form, site: { setting } } = this
    console.log('setting', setting)
    Object.keys(form).forEach((key: string) => {
      if (key === 'domain') {
        const protocolEndIndex = setting[key].indexOf('://')
        if (protocolEndIndex !== -1) {
          form[key] = setting[key].substring(protocolEndIndex + 3)
          this.protocol = setting[key].substring(0, protocolEndIndex + 3)
        }
      } else {
        form[key] = setting[key]
      }
    })

    if (form.privateKey) {
      this.remoteType = 'key'
    }
  }

  // /**
  //  * check form validate
  //  * @returns {boolean}
  //  */
  // checkFormValid() {
  //   // if (!['https://', 'http://'].some(d => this.form.domain.startsWith(d))) {
  //   //   this.$message.warn(this.$t('domainShouldStartsWithWarn'))
  //   //   return false
  //   // }
  //   return true
  // }

  submit() {
    // const formValid = this.checkFormValid()
    // if (!formValid) { return false }

    const form = {
      ...this.form,
      domain: `${this.protocol}${this.form.domain}`,
    }

    if (this.remoteType === 'password') {
      form.privateKey = ''
    } else {
      form.password = ''
    }

    ipcRenderer.send('setting-save', form)
    ipcRenderer.once('setting-saved', (event: IpcRendererEvent, result: any) => {
      this.$bus.$emit('site-reload')
      this.$message.success(this.$t('basicSettingSuccess'))

      ga.event('Setting', 'Setting - save', { evLabel: this.form.platform })
    })
  }

  async remoteDetect() {
    const form = {
      ...this.form,
      domain: `${this.protocol}${this.form.domain}`,
    }

    ipcRenderer.send('setting-save', form)
    ipcRenderer.once('setting-saved', () => {
      ipcRenderer.send('app-site-reload')
      ipcRenderer.once('app-site-loaded', () => {
        this.detectLoading = true
        ipcRenderer.send('remote-detect')

        ga.event('Setting', 'Setting - detect', { evLabel: this.form.platform })
        ipcRenderer.once('remote-detected', (event: IpcRendererEvent, result: any) => {
          console.log('检测结果', result)
          this.detectLoading = false
          if (result.success) {
            this.$message.success(this.$t('connectSuccess'))

            ga.event('Setting', 'Setting - detect-success', { evLabel: this.form.platform })
          } else {
            this.$message.error(this.$t('connectFailed'))

            ga.event('Setting', 'Setting - detect-failed', { evLabel: this.form.platform })
          }
        })
      })
    })
  }

  @Watch('form.token')
  onTokenChanged(val: string) {
    this.form.token = this.form.token.trim()
  }
}
</script>


<style lang="less" scoped>
.icon {
  cursor: pointer;
}
</style>
