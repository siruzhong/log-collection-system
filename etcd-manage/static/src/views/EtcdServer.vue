<style scoped>
.page{
    margin-top: 10px;
    text-align: right;
}
</style>

<template>
    <div>
        <Table border :columns="columns" :data="data1"></Table>
        <div class="page">
            <Page @on-change="changeListPage" @on-page-size-change="pageSizeChange" :total="total" :current="page" :page-size="pageSize" show-sizer />
        </div>
    </div>
</template>
<script>
export default {
    data() {
        return {
            page:1,
            total:0,
            pageSize:10,
            columns:[
                {
                    title: 'Title',
                    key: 'Title'
                },
                {
                    title: 'Name',
                    key: 'Name'
                },
                {
                    title: 'Address',
                    key: 'Address'
                },
                {
                    title: 'Roles',
                    key: 'Roles'
                },
                {
                    title: 'Desc',
                    key: 'Desc'
                },
                {
                    title: 'Action',
                    key: 'action',
                    width: 150,
                    align: 'center',
                    render: (h, params) => {
                        return h('div', [
                            h('Poptip', {
                                props: {
                                    confirm: true,
                                    title:this.$t('etcdServer.determineRepairDirectory')
                                },
                                on: {
                                    "on-ok": () => {
                                        this.restore(params.row);
                                    }
                                }
                            },[
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    }
                                }, this.$t('etcdServer.repairDirectory')),
                            ])
                        ]);
                    }
                }
            ],
            data:[],
            data1:[] // ?????????
        }
    },
    methods:{
        getList(){
            this.$http.get(`/v1/server`).then(response => {
                console.log(response);
                if(response.status == 200){
                    this.data = response.data || [];
                    this.total = this.data.length;
                    this.page = 1;
                    this.changeListPage(1);
                }
            }).catch(error=>{
                if (error.response){
                    this.$Message.error(error.response.data.msg);
                }
            });
        },
        // ????????????
        changeListPage(page){
            // pageSize
            this.data1.splice(0, this.data1.length);
            this.data1 = this.data.slice((page - 1) * this.pageSize, page * this.pageSize);
            console.log(page);
        },
        // ????????????
        pageSizeChange(pageSize){
            this.pageSize = pageSize;
            this.changeListPage(1);
            this.page = 1;
        },

        // ???????????????????????????
        restore(row){
            // console.log(row)
            this.$http.get(`/v1/restore`,{
                headers:{
                    "EtcdServerName":row.Name,
                }
            }).then(response=>{
                if(response.status == 200){
                    this.$Message.info('OK');
                }
            }).catch(error=>{
                if (error.response){
                    this.$Message.error(error.response.data.msg);
                }
            });
        }
    },
    mounted(){
        this.getList();
    }
}
</script>
