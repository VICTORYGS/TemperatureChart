<template>
    <el-form :model='form' class='demo-ruleForm' :rules='rules' ref='form' label-position='top'>

        <el-row :gutter='20' class='address'>
            <el-col :span='4'>
                <el-form-item prop='province'>
                    <el-select v-model='form.province' placeholder='请选择省' @change='proChange'>
                        <el-option v-for='item in provinces' :key='item.value' :value='item.value'>
                        </el-option>
                    </el-select>
                </el-form-item>
            </el-col>

            <el-col :span='4'>
                <el-form-item prop='city'>
                    <el-select v-model='form.city' placeholder='请选择市' @change='cityChange'>
                        <el-option v-for='item in citys' :key='item.value' :value='item.value'>
                        </el-option>
                    </el-select>
                </el-form-item>
            </el-col>

            <el-col :span='4'>
                <el-form-item prop='district'>
                    <el-select v-model='form.district' placeholder='请选择区/县' @change='districtChange'>
                        <el-option v-for='item in districts' :key='item.value' :value='item.value'>
                        </el-option>
                    </el-select>
                </el-form-item>
            </el-col>




        </el-row>
    </el-form>
</template>
<script>
    import addressData from '@/address'
    function formatData(data) {
        var result = [];
        for (var key in data) {
            result.push({
                value: key
            })
        }
        return result
    }
    function formatDistrictData(data) {
        var result = [];
        for (var i=0;i<data.length;i++) {
            result.push({
                value: data[i]
            })
        }
        return result
    }
    export default {
        name: 'addressInput',
        props: ['province', 'city', 'district', 'detail'],
        data: function () {
            return {
                rules: {
                    province: [{ required: true, message: '请选择省份', trigger: 'change' }],
                    city: [{ required: true, message: '请选择城市', trigger: 'change' }],
                    district: [{ required: true, message: '请选择区/县', trigger: 'change' }],
                    detail: [{ required: true, message: '请填写详细地址', trigger: 'change' }]
                },
                form: {
                    province: this.province,
                    city: this.city,
                    district: this.district,
                    detail: this.detail
                },
                provinces: formatData(addressData)
            }
        },
        watch: {
            form: {
                handler: function (val) {
                    this.$emit('change', val);
                },
                deep: true
            }
        },
        computed: {
            citys: function () {
                return formatData(addressData[this.form.province])
            },
            districts: function () {
                if(this.form.city){
                    return formatDistrictData(addressData[this.form.province][this.form.city])
                }
            }
        },
        methods: {
            proChange: function (val, oldVal) {
                if (oldVal) {
                    this.form.city = '';
                    this.form.detail = '';
                }
                var data = formatData(addressData[this.form.province]);
                for (var i = 0; i < data.length; i++) {
                    this.$set(this.citys, i, data[i]);
                }
                this.form.city = this.citys[0].value;
            },
            cityChange: function (val, oldVal) {
                this.form.district = this.districts[0].value;
            },
            districtChange: function (val, oldVal) {
                if (oldVal) {
                    this.form.detail = '';
                }
            },
            detailChange: function (val) {
                console.log(val);
            }
        }
    }
</script>
<style>
    .address .el-form-item {
        margin-bottom: 0!important;
        margin-right: 0!important;
    }
</style>