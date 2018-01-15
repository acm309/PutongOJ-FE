<template lang="html">
  <div>
    <Row type="flex" justify="start">
      <Col :span="2" class="label">Title</Col>
      <Col :span="21">
        <Input v-model="contest.title"></Input>
      </Col>
    </Row>
    <Row  type="flex" justify="start">
      <Col :span="2" class="label">Start Time</Col>
      <Col :span="8">
        <DatePicker
          v-model="contest.start"
          type="datetime"
          placeholder="选择日期时间">
        </DatePicker>
      </Col>
    </Row>
    <Row  type="flex" justify="start">
      <Col :span="2" class="label">End Time</Col>
      <Col :span="8">
        <DatePicker
          v-model="contest.end"
          type="datetime"
          placeholder="选择日期时间">
        </DatePicker>
      </Col>
    </Row>
    <Row type="flex" justify="start">
      <Col :span="2" class="label">Type</Col>
      <Col :span="4">
        <Select v-model="contest.encrypt" placeholder="请选择">
          <Option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </Option>
        </Select>
      </Col>
    </Row>
    <Row v-if="contest.encrypt === 2">
      <Transfer
        :data="transData"
        :target-keys="targetKeys"
        :list-style="listStyle"
        :operations="['To left','To right']"
        filterable
        :filter-method="filterMethod"
        @on-change="handleChange">
        <div :style="{float: 'right', margin: '5px'}">
          <Button type="ghost" size="small" @click="">Refresh</Button>
        </div>
    </Transfer>
    </Row>
    <Row v-if="contest.encrypt === 3">
      <Col :span="23">
        <Input v-model="contest.argument"></Input>
      </Col>
    </Row>
    <Row>
      <Col :span="23"><hr></Col>
    </Row>
    <Row>
      <Col :span="23">
        <draggable v-model="contest.list">
          <transition-group name="list">
            <div v-for="(item, index) in contest.list" :key="index" class="list-item">
              <div>{{ item }} -- {{ jobs[item] }}</div>
              <Icon type="close-circled" @click.native="removeJob(index)"></Icon>
            </div>
          </transition-group>
        </draggable>
      </Col>
    </Row>
    <Row>
      <Col :span="21" class="add">
        <Input v-model="pid" placeholder="Add a pid" @keyup.enter.native="add"></Input>
      </Col>
      <Col :span="2">
        <Button type="primary" @click="add">Add</Button>
      </Col>
    </Row>
    <Row>
      <Col :span="23"><hr></Col>
    </Row>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import only from 'only'
import { mapGetters } from 'vuex'

export default {
  data () {
    return {
      jobs: {},
      options: [
        {
          value: 1,
          label: 'Public'
        },
        {
          value: 2,
          label: 'Private'
        },
        {
          value: 3,
          label: 'Password'
        }
      ],
      pid: '',
      targetKeys: [],
      listStyle: {
        width: '400px',
        height: '500px'
      }
    }
  },
  props: ['contest', 'overview'],
  computed: {
    ...mapGetters({
      list: 'user/list'
    }),
    transData () {
      let data = []
      this.list.forEach((item, index) => {
        data.push({
          key: index + 1 + '',
          label: item.uid,
          disabled: true
        })
      })
      return data
    }
  },
  created () {
    // 这里必须保证此时 overview 是存在的
    // 如果用户没有点过 overview tab 时，就会出现 overview 不存在的情况
    if (this.$route.params.cid) {
      let p = Promise.resolve()
      if (this.overview.length === 0) {
        p = this.$store.dispatch('contest/findOne', only(this.$route.params, 'cid'))
      }
      p.then(() => {
        this.overview.forEach((item) => {
          // https://vuejs.org/2016/02/06/common-gotchas/
          this.$set(this.jobs, item.pid, item.title)
        })
      })
    }
    this.$store.dispatch('user/find')
  },
  methods: {
    add () {
      this.$store.dispatch('problem/findOne', only(this, 'pid'))
        .then(({ problem }) => {
          this.contest.list.push(problem.pid)
          this.$set(this.jobs, problem.pid, problem.title)
          this.pid = ''
        })
    },
    removeJob (index) {
      this.contest.list.splice(index, 1)
    },
    filterMethod (data, query) {
      // return data.label.indexOf(query) > -1
    },
    handleChange (newTargetKeys) {
      this.targetKeys = newTargetKeys
    }
  },
  components: {
    draggable
  }
}
</script>

<style lang="stylus" scoped>
.ivu-row, .ivu-row-flex
  margin-bottom: 14px
.ivu-col
  text-align: left
  font-size: 16px
.label
  line-height: 32px
  hr
    background-color: #dbdbdb
    border: none
    height: 1px
  .ivu-btn
    margin-left: 20px
.list-item
  display: flex
  justify-content: space-between
  padding: 14px 20px
  margin-bottom: 14px
  background-color: #f2f2f2
.ivu-icon-close-circled
  line-height: 20px
  color: #c3c2c2
  cursor: pointer
  &:hover
    font-size: 20px
.add
  margin-bottom: 20px
</style>