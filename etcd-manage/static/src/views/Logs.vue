<style scoped>
.logs{
  width: 100%;
  height: 100vh;
  overflow-y: scroll;
  overflow-x: hidden;
}
.search{
    width: 100%;
    height: 39px;
}
</style>

<template>
    <div class="logs">
        <div class="search">
            <Form :rules="ruleInline" inline>
                <FormItem :prop="$t('logs.date')">
                    <DatePicker type="date" placeholder="Select date" style="width: 300px" @on-change="changeDate" format="yyyyMMdd"></DatePicker>
                </FormItem>
                <FormItem :prop="$t('logs.user')">
                    <Select v-model="user" clearable style="width:200px">
                        <Option v-for="item in users" :value="item.name" :key="item.name">
                            {{ item.name }}
                            <!-- <span>{{ item.name }}</span>
                            <span style="float:right;color:#ccc">{{ item.role }}</span> -->
                        </Option>
                    </Select>
                </FormItem>
                <FormItem :prop="$t('logs.typeof')">
                    <Select v-model="logType" clearable style="width:200px">
                        <Option v-for="item in logtypes" :value="item" :key="item">{{ item }}</Option>
                    </Select>
                </FormItem>
                <FormItem>
                    <Button type="primary" @click="getList">{{$t('logs.filter')}}</Button>
                </FormItem>
            </Form>
        </div>
        <Table border :columns="columns" :data="list" :loading="loading"></Table>
        <div style="margin-top:10px; text-align: right;">
            <Page @on-change="changeListPage" @on-page-size-change="pageSizeChange" show-sizer :current="page" :page-size="pageSize" :total="listTotal" />
        </div>
        <div style="height:200px;"></div>
    </div>
</template>

<script>
export default {
        data(){
        return {
            list:[],
            page:1,
            pageSize:10,
            listTotal:0,
            loading:false,
            date:'',
            logtypes:[], // ??????????????????
            users:[], // ????????????
            logType:'', // ????????????
            user:'', // ?????????
            columns:[{
                    title: 'Date',
                    key: 'date'
                },
                {
                    title: 'User',
                    key: 'user'
                },
                {
                    title: 'Role',
                    key: 'role'
                },
                {
                    title: 'Action',
                    key: 'msg'
            }]
        }
    },
    mounted(){
        this.bindDate = new Date();
        // this.getList();
        this.$Message.info(this.$t('logs.selectDateShowLog'));

        // ???????????????
        this.getUsers();
        this.getLogtypes();
    },
    methods:{
        // ????????????????????????
        pageSizeChange(pageSize){
            this.pageSize = pageSize;
            this.page = 1;
            this.getList();
        },

        // ????????????
        changeListPage(page){
            this.page = page;
            this.getList();
        },

        // ????????????
        changeDate(date){
            console.log(date);
            this.date = date;
            this.page = 1;
            this.getList();
        },

        // ????????????
        getList(){
            if (this.date == '' || !this.date){
                this.$Message.info(this.$t('logs.selectDate'));
                return
            }
            this.loading = true;
            this.$Loading.start();
            this.list = [];
            this.listTotal = 0;

            this.user = this.user || '';
            this.logType = this.logType || '';

            this.$http.get(`/v1/logs?date=${this.date}&page=${this.page}&page_size=${this.pageSize}&user=${this.user}&log_type=${this.logType}`)
            .then(response => {
                console.log(response);
                if(response.status == 200){
                    this.list = response.data.list;
                    this.listTotal = response.data.total;
                }
                this.$Loading.finish();
                this.loading = false;
            }).catch(error=>{
                if (error.response){
                    this.$Message.error(error.response.data.msg);
                }
                this.loading = false;
                this.$Loading.error();
            });
        },

        // ??????????????????
        getUsers(){
            this.$http.get(`/v1/users`)
            .then(response => {
                console.log(response);
                if(response.status == 200){
                    this.users = response.data || [];
                }
            }).catch(error=>{
                if (error.response){
                    this.$Message.error(error.response.data.msg);
                }
            });
        },

        // ????????????????????????
        getLogtypes(){
            this.$http.get(`/v1/logtypes`)
            .then(response => {
                console.log(response);
                if(response.status == 200){
                    this.logtypes = response.data || [];
                }
            }).catch(error=>{
                if (error.response){
                    this.$Message.error(error.response.data.msg);
                }
            });
        }
    }
}
</script>
