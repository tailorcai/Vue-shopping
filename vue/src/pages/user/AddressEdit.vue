<template>
<transition name='bounce2'>
    <!-- 新增和编辑地址 -->
    <div class="address-edit">
        <van-nav-bar
            title="编辑地址"
            left-arrow
            @click-left="goBack"
        />
        <div>
            <van-address-edit
                :area-list="areaList"
                :show-delete='showDelete'
                show-set-default
                show-search-result
                :addressInfo='addressInfo'
                :search-result="searchResult"
                @save="onSave"
                @delete="onDelete"
            />
        </div>
    </div>
</transition>  
</template>

<script>
import {mapGetters,mapMutations,mapActions} from 'vuex'
import {Toast} from 'vant'
export default {
    computed: {
        ...mapGetters(['addressInfo'])
    },

    data() {
        return {
            searchResult: [],
            areaList: require('js/area.js').default,
            showDelete: false
        }
    },

    created() {
        if (this.addressInfo) {
            this.showDelete = true
        }
    },
    methods: {
        async onSave(val) {
            // 以下参数在api接口查看详情
            try {
                const {data} = await this.Api.postAddress({
                        name: val.name,
                        tel: val.tel,
                        address: val.province + val.city + val.county + val.addressDetail,
                        isDefault: val.isDefault,
                        province: val.province,
                        city: val.city,
                        county: val.county,
                        addressDetail: val.addressDetail,
                        areaCode: val.areaCode,
                        id: this.addressInfo ? this.addressInfo._id : undefined  // 修改地址时候要传id
                })
                    if (data.code == 200) {
                        this.Toast(data.msg);
                        setTimeout(() => {
                            this.$router.go(-1)
                            this.clearAddress('')
                        }, 1000);
                    } else {
                        this.$router.push('/user/login')
                    }
                
            } catch (error) {
                this.Toast('网络错误')
            }
            
        },
        async onDelete(val) {
            try {
                const {data} = await this.Api.deleteAddress(val._id)
                if (data.code == 200) {
                    this.Toast(data.msg);
                    setTimeout(() => {
                        this.$router.go(-1)
                        this.clearAddress('')
                    }, 1000);
                }else {
                    this.$router.push('/user/login')
                }
            } catch (error) {
                this.Toast('网络错误')
            }
        },
        
        goBack() {
            this.$router.go(-1)
            setTimeout(() => {
                this.clearAddress('')
            }, 300);
        },

        ...mapActions(['clearAddress']),

        
    }
}
</script>

<style lang="stylus" scoped>
.address-edit
    position fixed
    top 0px
    left 0
    right 0
    bottom 0
    z-index 200
    background #fff
.bounce2-enter-active {
    animation: bounce-in .3s;
        }
    .bounce2-leave-active {
            animation: bounce-in .3s reverse;
            }
    @keyframes bounce-in {
    0% {
        transform: translate3d(-100%,0,0)
        }
    
    100% {
        transform: translate3d(0,0,0)
        }
}     
</style>