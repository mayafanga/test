<template>
  <section class="app-container">
    <el-tooltip class="item" effect="dark" content="点击预览">
      <el-button class="previewBtn" type="text" size="medium" @click="showPreviewLink">
        <i class="el-icon-view"></i>
        <br />
        <span style="fontSize: 14px;">预览</span>
      </el-button>
    </el-tooltip>
    <section class="form-container">
      <el-row style="margin-bottom: 1rem">
        <span
          style="font-weight: bolder; display: inline-block; width: 8.5rem; text-align: right"
        >基本信息</span>
      </el-row>
      <el-form
        ref="actForm"
        :model="activity"
        label-width="10rem"
        :rules="activityRules"
        label-suffix="："
      >
        <el-form-item label="活动名称" prop="activityName">
          <el-input v-model="activity.activityName" placeholder="请输入活动名称" />
        </el-form-item>
        <el-form-item label="报名时间" prop="signDateRange">
          <el-date-picker
            v-model="activityExt.signDateRange"
            start-placeholder="请选择活动开始时间"
            end-placeholder="请选择活动结束时间"
            type="daterange"
            value-format="timestamp"
            size="medium"
            style="width: 100%"
            @change="setSignDateRange"
          />
        </el-form-item>
        <el-form-item label="活动时间" prop="dateRange">
          <el-date-picker
            v-model="activityExt.dateRange"
            start-placeholder="请选择活动开始时间"
            end-placeholder="请选择活动结束时间"
            type="daterange"
            value-format="timestamp"
            size="medium"
            style="width: 100%"
            @change="setDateRange"
          />
        </el-form-item>
        <el-form-item ref="coverUrl" label="封面banner" prop="coverUrl">
          <upload-img @upFinish="setCoverBanner" :value="activity.coverUrl" />
          <p class="tips">提示：请上传690×300尺寸的图片，大小不能超过2M。</p>
        </el-form-item>
        <el-form-item ref="detailUrl" label="详情页banner" prop="detailUrl">
          <upload-img @upFinish="setDetailBanner" :value="activity.detailUrl" />
          <p class="tips">提示：请上传690×300尺寸的图片，大小不能超过2M。</p>
        </el-form-item>
        <el-form-item label="活动简介" prop="content">
          <tinymce v-model="activity.content" :height="300" />
        </el-form-item>
        <el-row style="margin-bottom: 1rem">
          <span
            style="font-weight: bolder; display: inline-block; width: 8.5rem; text-align: right"
          >报名信息</span>
        </el-row>
        <el-form-item label="报名审核" prop="isAudit">
          <el-radio-group v-model="activity.isAudit" :disabled="isUpd">
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="报名资料" prop="enterOpt">
          <el-checkbox-group v-model="activityExt.realOpt" :disabled="true">
            <el-checkbox
              v-for="i in activityExt.realChkOpt"
              :key="i.value"
              :label="i.value"
            >{{ i.label }}</el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-form-item label="其他填写项" prop="otherOpt">
          <el-radio-group v-model="activityExt.hasOtherOpt" :disabled="isUpd">
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
          <el-button
            v-show="!(isUpd || !activityExt.hasOtherOpt)"
            style="float: right"
            size="small"
            @click="addOtherOpt"
          >添加信息</el-button>
        </el-form-item>
        <el-table
          v-if="activityExt.hasOtherOpt"
          :data="activityExt.otherOpt"
          style="margin-bottom: 1rem"
          border
          fit
        >
          <el-table-column label="序号" align="center" width="60">
            <template slot-scope="{$index}">{{ $index + 1 }}</template>
          </el-table-column>
          <el-table-column label="类型" align="center">
            <template slot-scope="{row}">
              <el-select
                v-model="row.type"
                :disabled="!isUpd && !row.isEdit"
                @change="syncCurrentOptIntoOtherOpt(row)"
              >
                <el-option
                  v-for="item in activityExt.selectOpt"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                />
              </el-select>
            </template>
          </el-table-column>
          <el-table-column label="标题" align="center">
            <template slot-scope="{row}">
              <el-input
                v-model.trim="row.labelName"
                :disabled="!isUpd && !row.isEdit"
                @input="syncCurrentOptIntoOtherOpt(row)"
              />
            </template>
          </el-table-column>
          <el-table-column label="描述" align="center">
            <template slot-scope="{row}">
              <el-input
                v-model="row.desc"
                placeholder="请输入信息"
                :disabled="!isUpd && !row.isEdit"
                @input="syncCurrentOptIntoOtherOpt(row)"
              />
            </template>
          </el-table-column>
          <el-table-column label="预设选项" align="center">
            <template slot-scope="{row}">
              <el-input
                v-if="row.type === 3"
                v-model.trim="row.extendsJsonStr"
                :disabled="isUpd"
                placeholder="预设选项用逗号分隔"
                @input="syncCurrentOptIntoOtherOpt(row)"
              />
              <span v-else>--</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" align="center">
            <template slot-scope="{row}">
              <el-button type="text" :disabled="!isUpd && !row.isEdit" @click="delOtherOpt(row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-row style="margin-bottom: 1rem">
          <span
            style="font-weight: bolder; display: inline-block; width: 8.5rem; text-align: right"
          >团队报名</span>
        </el-row>
        <el-form-item label="团队报名" prop="isTeam">
          <el-radio-group v-model="activity.isTeam" :disabled="isUpd" @change="changeSignMode">
            <el-radio :label="false">否</el-radio>
            <el-radio :label="true">是</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item v-if="activity.isTeam" label="成员数量" prop="teamMember">
          <el-radio-group v-model="activity.teamMember" :disabled="isUpd">
            <el-radio :label="2">2人</el-radio>
            <el-radio :label="3">3人</el-radio>
            <el-radio :label="4">4人</el-radio>
            <el-radio :label="5">5人</el-radio>
          </el-radio-group>
        </el-form-item>
        <p
          v-if="activity.teamMember !== 1"
          style="color: #4fa2ff; padding-left: 4.7rem"
        >备注：队长不记入成员人数</p>
        <el-form-item label="团队名称为医院" prop="isHospital">
          <el-radio-group v-model="activity.isHospital" :disabled="isUpd">
            <el-radio :label="false">否</el-radio>
            <el-radio :label="true">是</el-radio>
          </el-radio-group>
        </el-form-item>
        <p style="color: #4fa2ff; padding-left: 4.7rem">备注：当名称为医院时，医院名称只能从“医院名称字典”中选择</p>
        <el-form-item>
          <el-button type="primary" :loading="saveLoading" @click="cuAct">保存</el-button>
        </el-form-item>
      </el-form>
    </section>
    <el-dialog :visible.sync="dialogPreviewVisible" width="700px" title="预览链接">
      <preview-link :preview-list="previewList" />
    </el-dialog>
  </section>
</template>

<script>
import PreviewLink from '@/components/PreviewLink'
import upload from '@/components/upload'
import UploadImg from '@/components/UploadImg'
import tinymce from '@/components/Tinymce'
import { activityPreview } from '@/api/preview'
import { creActivity, getActivity, updActivity } from '@/api/activity'
import constantObj from '@/dictData/constant'

export default {
  name: 'CreActivity',
  components: { upload, tinymce, PreviewLink, UploadImg },
  data () {
    return {
      saveLoading: false,
      uuid: '',
      dialogPreviewVisible: false,
      isUpd: false,
      previewList: [],
      activity: {
        activityName: '',
        coverUrl: 'https://zsm-1301491369.cos.ap-beijing.myqcloud.com/img/66f776b03f0c6f7e44d0b755f3e22ab7.png',
        detailUrl: 'https://zsm-1301491369.cos.ap-beijing.myqcloud.com/img/709d11cbfb1e6cb052b6d709cee99aa7.png',
        isTeam: true,
        teamMember: 2,
        isHospital: true,
        isReal: true,
        isAudit: true,
        startTime: null,
        endTime: null,
        signStartTime: null,
        signEndTime: null,
      },
      activityExt: {
        realOpt: [1, 2, 3, 4, 5, 6],
        realChkOpt: [
          { value: 1, label: '姓名' },
          { value: 2, label: '手机号' },
          { value: 3, label: '职业' },
          { value: 4, label: '单位' },
          { value: 5, label: '地区' },
          { value: 6, label: '科室' }
        ],
        dateRange: null,
        signDateRange: null,
        hasOtherOpt: false,
        defaultOtherOpt: {
          id: 1,
          labelName: '文本',
          labelValue: '',
          desc: '',
          type: 1,
          extendsJsonStr: '',
          extendsJsonList: [],
          isEdit: true
        },
        otherOpt: [
          {
            id: 1,
            labelName: '姓名',
            labelValue: '',
            desc: '',
            type: 1,
            extendsJsonStr: '',
            extendsJsonList: [],
            isExtend: false,
            isEdit: false
          },
          {
            id: 2,
            labelName: '单位',
            labelValue: '',
            desc: '',
            type: 1,
            extendsJsonStr: '',
            extendsJsonList: [],
            isExtend: false,
            isEdit: false
          },
          {
            id: 3,
            labelName: '单位所在地区',
            labelValue: '',
            desc: '',
            type: 1,
            extendsJsonStr: '',
            extendsJsonList: [],
            isExtend: false,
            isEdit: false
          },
          {
            id: 4,
            labelName: '职业',
            labelValue: '',
            desc: '',
            type: 1,
            extendsJsonStr: '',
            extendsJsonList: [],
            isExtend: false,
            isEdit: false
          },
          {
            id: 5,
            labelName: '联系电话',
            labelValue: '',
            desc: '',
            type: 1,
            extendsJsonStr: '',
            extendsJsonList: [],
            isExtend: false,
            isEdit: false
          },
          {
            id: 6,
            labelName: '科室',
            labelValue: '',
            desc: '',
            type: 1,
            extendsJsonStr: '',
            extendsJsonList: [],
            isExtend: false,
            isEdit: false
          },
          {
            id: 7,
            labelName: '文本',
            labelValue: '',
            desc: '',
            type: 1,
            extendsJsonStr: '',
            extendsJsonList: [],
            isExtend: false,
            isEdit: true
          }
        ],
        selectOpt: [
          { value: 1, label: '文本' },
          { value: 2, label: '数字' },
          { value: 3, label: '下拉选择' }
        ]
      },
      activityRules: {
        activityName: [
          { required: true, message: '请输入活动名称', trigger: 'blur' }
        ],
        signDateRange: [
          { required: true, validator: this.chkSignDateRangeValid, trigger: 'blur' }
        ],
        dateRange: [
          { required: true, validator: this.chkDateRangeValid, trigger: 'blur' }
        ],
        coverUrl: [
          { required: true, message: '请上传封面banner' }
        ],
        detailUrl: [
          { required: true, message: '请上传详情页banner' }
        ]
      }
    }
  },
  watch: {
    'activity.detailUrl' (v) {
      if (v) {
        this.$refs.detailUrl.clearValidate()
      }
    },
    'activity.coverUrl' (v) {
      console.log(v)
      if (v) {
        this.$refs.coverUrl.clearValidate()
      }
    }
  },
  mounted () {
    let id = this.$route.query.id
    const act = this.$getSession('currentAct')
    if (id) this.isUpd = true
    else {
      if (act) {
        id = act.id
        this.isUpd = true
      }
    }
    if (this.isUpd) {
      if (!act || act.id !== Number.parseInt(id)) {
        getActivity(id).then(r => {
          this.initActData(r.data)
          this.$setSession('currentAct', r.data)
        })
      } else {
        this.initActData(act)
      }
    }
  },
  methods: {
    changeSignMode (val) {
      this.activity.teamMember = val ? 3 : 1
    },
    showPreviewLink () {
      this.buildSendData()
      this.activity = this.uuid ? { ...this.activity, uuid: this.uuid } : this.activity
      activityPreview(this.activity).then(res => {
        const uuid = res.data
        this.uuid = uuid
        const baseUrl = process.env.VUE_APP_BASE_API === 'https://api.zhishimao.com' ? 'https://zhishimao.com' : 'https://tm.zhishimao.com'
        const address1 = baseUrl + '/#/preview/activityList' + '?uuid=' + uuid
        const address2 = baseUrl + '/#/preview/activityPreDetail' + '?uuid=' + uuid
        this.previewList = [
          {
            name: 'activityList',
            label: '活动列表',
            address: address1
          },
          {
            name: 'activityPreDetail',
            label: '活动详情',
            address: address2
          }
        ]
        this.dialogPreviewVisible = true
      })
    },
    initActData (data) {
      this.activity.id = data.id
      this.activity.activityName = data.activityName
      this.activity.startTime = data.startTime
      this.activity.endTime = data.endTime
      this.activity.signStartTime = data.signStartTime
      this.activity.signEndTime = data.signEndTime
      this.activityExt.dateRange = [data.startTime, data.endTime]
      this.activityExt.signDateRange = [data.signStartTime, data.signEndTime]
      this.activity.coverUrl = data.coverUrl
      this.activity.detailUrl = data.detailUrl
      this.activity.content = data.content
      this.activity.isAudit = data.isAudit
      this.activity.isReal = data.isReal
      this.activity.isTeam = data.isTeam
      this.activity.teamMember = data.teamMember
      this.activity.isHospital = data.isHospital

      if (data.extendsJson.length) {
        const extJson = JSON.parse(data.extendsJson)
        this.activityExt.otherOpt = []
        extJson.forEach((value, index) => {
          if (value.isExtend) {
            const v = Number.parseInt(value.desc)
            if (this.activityExt.realOpt.indexOf(v) === -1) this.activityExt.realOpt.push(v)
          } else {
            value.id = index + 1
            if (value.extendsJsonList) {
              value.extendsJsonStr = value.extendsJsonList.join(',')
            }
            this.activityExt.otherOpt.push(value)
            if (this.activityExt.otherOpt.length) this.activityExt.hasOtherOpt = true
          }
        })
      }

      this.activityExt.hasOtherOpt = !!this.activityExt.otherOpt.length
    },
    chkDateRangeValid (rule, value, callback) {
      if (!this.activity.startTime) {
        callback(new Error('请选择活动起止时间'))
      } else {
        callback()
      }
    },
    chkSignDateRangeValid (rule, value, callback) {
      if (!this.activity.signEndTime) {
        callback(new Error('请选择活动报名起止时间'))
      } else {
        callback()
      }
    },
    setCoverBanner (obj) {
      this.activity.coverUrl = obj.url
    },
    setDetailBanner (obj) {
      this.activity.detailUrl = obj.url
    },
    cuAct () {
      console.log(this.activityExt.otherOpt, 'this.activityExt.otherOpt')
      return false
      this.$refs['actForm'].validate((valid) => {
        if (!valid) {
          this.$message.warning('请填写完整')
          return false
        }
        if (this.activity.startTime < this.activity.signStartTime) {
          this.$message.warning('活动报名开始时间不可小于活动开始时间')
          return false
        }
        if (this.activity.signEndTime > this.activity.endTime) {
          this.$message.warning('活动报名结束时间不可大于活动结束时间')
          return false
        }
        this.saveLoading = true
        this.buildSendData()
        if (this.isUpd) {
          updActivity(this.activity).then(() => {
            this.$setSession('currentAct', this.activity)
            this.$message.success('更新成功')
            this.saveLoading = false
            this.$router.push({ name: 'myActivityList' })
          }).catch(() => {
            this.saveLoading = false
          })
        } else {
          creActivity(this.activity).then(r => {
            this.$message.success('新建成功')
            this.saveLoading = false
            this.$router.push({ name: 'myActivityList' })
          }).catch(() => {
            this.saveLoading = false
          })
        }
      })
    },
    buildSendData () {
      const extJson = []
      if (this.activityExt.hasOtherOpt) {
        const other = [...this.activityExt.otherOpt]
        other.forEach(value => {
          delete value.id
          if (value.extendsJsonStr) {
            const t = value.extendsJsonStr.replace(/，/g, ',').split(',').filter(v => {
              return v.length
            })

            value.extendsJsonList = []
            t.forEach((v) => {
              value.extendsJsonList.push(v)
            })
          }
          delete value.extendsJsonStr
          extJson.push(value)
        })
      }

      this.activity.extendsJson = JSON.stringify(extJson)
      this.activity.storeId = this.$getSession('currentShop').id
      if (this.isUpd) {
        let id = this.$route.query.id
        if (id === undefined) id = this.activity.id
        this.activity.id = id
      }
    },
    setSignDateRange (val) {
      if (val) {
        this.activity.signStartTime = val[0]
        this.activity.signEndTime = val[1] + 3600000 * 24 - 1
      } else {
        this.activity.signStartTime = null
        this.activity.signEndTime = null
      }
    },
    setDateRange (val) {
      if (val) {
        this.activity.startTime = val[0]
        this.activity.endTime = val[1] + 3600000 * 24 - 1
      } else {
        this.activity.startTime = null
        this.activity.endTime = null
      }
    },
    syncCurrentOptIntoOtherOpt (opt) {
      console.log(opt, 'optopt')
      for (const d of this.activityExt.realChkOpt) {
        if (d.label === opt.labelName) {
          this.$message.warning('本项标题在活动报名必填中已存在')
          break
        }
      }
      let has = this.getOtherOptById(opt.id)
      has = opt
    },
    addOtherOpt () {
      let id = this.activityExt.defaultOtherOpt.id
      this.activityExt.otherOpt.forEach(value => {
        id = id < value.id ? value.id : id
      })
      id++

      const has = this.getOtherOptById(id)
      if (!has.id) {
        const newOpt = { ...this.activityExt.defaultOtherOpt }
        newOpt.id = id
        this.activityExt.otherOpt.push(newOpt)
      }
    },
    delOtherOpt (opt) {
      const id = opt.id
      this.activityExt.otherOpt = this.activityExt.otherOpt.filter(value => {
        return value.id !== id
      })
    },
    getOtherOptById (id) {
      let res = {}
      for (const value of this.activityExt.otherOpt) {
        if (value.id === id) {
          res = value
          break
        }
      }
      return res
    }
  }
}
</script>

<style scoped>
.form-container {
  width: 1000px;
  margin: 0 auto;
}
</style>
