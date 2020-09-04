<template>
    <div>
        <head-top></head-top>
        <el-row style="margin-top: 20px;">
  			<el-col :span="14" :offset="4">
	  			<header class="form_header">借阅图书</header>
		        <el-form :model="formData" :rules="rules" ref="formData" label-width="110px" class="demo-formData">
					<el-form-item label="图书ID" prop="book_id">
						<el-input v-model="formData.book_id"></el-input>
					</el-form-item>
                    <el-form-item label="借阅日期" prop="time_slot_id">
                     <el-date-picker
                         v-model="formData.time_slot_id"
                         align="right"
                         type="date"
                         placeholder="选择日期"
                         :picker-options="pickerOptions">
                     </el-date-picker>
                    </el-form-item>
                    <el-form-item label="预计借阅时长（天）" prop="time_cost">
                        <el-slider v-model="formData.time_cost" :format-tooltip="formatTooltip"
                                   range
                                   show-stops
                                   :max="31">
                        </el-slider>
                        <span>天数：{{formData.time_cost}}</span>
                    </el-form-item>
					<el-form-item class="button_submit">
						<el-button type="primary" @click="submitForm('formData')">立即创建</el-button>
					</el-form-item>
				</el-form>
  			</el-col>
  		</el-row>
    </div>
</template>

<script>
    import headTop from '@/components/headTop'
    import {borrows,bookCategory} from '@/api/getData'
    import {baseUrl, baseImgPath} from '@/config/env'
    export default {
    	data(){
    		return {
    			city: {},
    			formData: {
					book_id: '', //图书ID
					time_slot_id: '', //时间戳
                    borrow_id: '',
                    reader_id: '',
                    renew:2,
		        },
		        rules: {
					book_id: [
						{ required: true, message: '请输入书籍名称', trigger: 'blur' },
					],
					time_slot_id: [
						{ required: true, message: '请输入借阅日期' },
					],
					time_cost: [
						{ required: true, message: '请输入预计的借阅时间' },
					],
				},
			    baseUrl,
    		}
    	},
    	components: {
    		headTop,
    	},
    	mounted(){
    		this.initData();
    	},
    	methods: {
    		async initData(){
    			try{
    				this.city = await cityGuess();
    				const categories = await foodCategory();
    				categories.forEach(item => {
    					if (item.sub_categories.length) {
    						const addnew = {
    							value: item.book_id,
						        label: item.book_id,
						        children: []
    						}
    						item.sub_categories.forEach((subitem, index) => {
    							if (index == 0) {
    								return
    							}
    							addnew.children.push({
    								value: subitem.book_id,
						        	label: subitem.book_id,
    							})
    						})
    						this.categoryOptions.push(addnew)

    					}
    				})
    			}catch(err){
    				console.log(err);
    			}
    		},  
		    handleDelete(index){
		    	this.activities.splice(index, 1)
		    },
		    submitForm(formbook_id) {
				this.$refs[formbook_id].validate(async (valid) => {
					if (valid) {
						Object.assign(this.formData, {activities: this.activities}, {
							category: this.selectedCategory.join('/')
						})
						try{
							let result = await borrows(this.formData);
							if (result.status == 1) {
								this.$message({
					            	type: 'success',
					            	message: '添加成功'
					          	});
					          	this.formData = {
                                    book_id: '', //书籍名称
									time_slot_id: '', //时间戳
                                    time_cost: '',
                                    renew:2,
						        };
                            }
                            else {
								this.$message({
					            	type: 'error',
					            	message: result.message
					          	});
							}
							console.log(result)
						}catch(err){
							console.log(err)
						}
					} else {
						this.$notify.error({
							title: '错误',
							message: '请检查输入是否正确',
							offset: 100
						});
						return false;
					}
				});
			},
		}
    }
</script>

<style lang="less">
	@import '../style/mixin';
	.button_submit{
		text-align: center;
	}
    .form_header {
        text-align: center;
        margin-bottom: 30px;
    }
	.avatar-uploader .el-upload {
	    border: 1px dashed #d9d9d9;
	    border-radius: 6px;
	    cursor: pointer;
	    position: relative;
	    overflow: hidden;
	}
	.avatar-uploader .el-upload:hover {
	    border-color: #20a0ff;
	}
	.avatar-uploader-icon {
	    font-size: 28px;
	    color: #8c939d;
	    width: 120px;
	    height: 120px;
	    line-height: 120px;
	    text-align: center;
	}
	.avatar {
	    width: 120px;
	    height: 120px;
	    display: block;
	}
	.el-table .info-row {
	    background: #c9e5f5;
	}

	.el-table .positive-row {
	    background: #e2f0e4;
	}
</style>
