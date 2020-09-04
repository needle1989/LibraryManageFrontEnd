<template>
    <div>
     	<head-top></head-top>
        <el-row style="margin-top: 20px;">
  			<el-col :span="14" :offset="4">
  				<header class="form_header">归还书籍</header>
	  			<el-form :model="recommendForm" ref="recommendForm" label-width="110px" class="form">
		  			<el-row class="category_select">
		  				<el-form-item label="是否推荐阅读">
			  				<el-select v-model="recommendForm.categorySelect" :placeholder="selectValue.label" style="width:100%;">
							    <el-option
							      	v-for="item in recommendForm.categoryList"
							      	:key="item.value"
							      	:label="item.label"
							      	:value="item.value">
							    </el-option>
						  	</el-select>
						</el-form-item>
					</el-row>
					<el-row class="add_category_row" :class="showAddCategory? 'showEdit': ''">
						<div class="add_category">
							<el-form-item label="推荐理由" prop="description">
								<el-input v-model="recommendForm.description"></el-input>
							</el-form-item>
							<el-form-item>
								<el-button type="primary" @click="submitrecommendForm('recommendForm')">提交</el-button>
							</el-form-item>
						</div>
					</el-row>
					<div class="add_category_button" @click="addCategoryFun">
						<i class="el-icon-caret-top edit_icon" v-if="showAddCategory"></i>
						<i class="el-icon-caret-bottom edit_icon" v-else slot="icon"></i>
						<span>推荐阅读表</span>
					</div>
	  			</el-form>
	  			<header class="form_header">归还书籍</header>
	  			<el-form :model="returnForm" :rules="returnrules" ref="returnForm" label-width="110px" class="form return_form">
	  				<el-form-item label="书籍ID" prop="book_id">
						<el-input v-model="returnForm.book_id"></el-input>
					</el-form-item>
					<el-form-item label="借阅日期" prop="time_slot_id">
						<el-input v-model="returnForm.time_slot_id"></el-input>
					</el-form-item>
					<el-form-item>
						<el-button type="primary" @click="returns('returnForm')">确认归还</el-button>
					</el-form-item>
	  			</el-form>
  			</el-col>
  		</el-row>
    </div>
</template>

<script>
import headTop from '@/components/headTop'
    import {getCategory, addCategory, returns} from '@/api/getData'
    import {baseUrl} from '@/config/env'
    export default {
    	data(){
    		return {
    			baseUrl,
    			borrow_id: 1,
    			recommendForm: {
    				categoryList: [],
    				categorySelect: '',
    				reader_id: '',
    				description: '',
    			},
    			returnForm: {
    				book_id: '',
    				description: '',
    				time_slot_id: '',
    				attributes: [],
    			},
    			returnrules: {
                    book_id: [
						{ required: true, message: '请输入书籍ID', trigger: 'blur' },
                    ],
                    time_slot_id: [
                        { required: true, message: '请输入借阅日期', trigger: 'blur' },
                    ],
    			},
    		}
    	},
    	components: {
    		headTop,
    	},
    	created(){
            if (this.$route.query.borrow_id) {
                this.borrow_id = this.$route.query.borrow_id;
    		}else{
                this.borrow_id = Math.ceil(Math.random()*10);
    			this.$msgbox({
		          title: '提示',
		          message: '归还书籍需要有未归还的书籍，先去借阅书籍吗？',
		          showCancelButton: true,
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          beforeClose: (action, instance, done) => {
		            if (action === 'confirm') {
		              this.$router.push('/borrows');
		              done();
		            } else {
		            	this.$message({
				            type: 'info',
				            message: '取消'
				        });
		              	done();
		            }
		          }
		        })
    		}
    		this.initData();
    	},
    	computed: {
    		selectValue: function (){
    			return this.recommendForm.categoryList[this.recommendForm.categorySelect]||{}
    		}
    	},
    	methods: {
    		async initData(){
    			try{
    				const result = await getCategory(this.borrow_id);
	    			if (result.status == 1) {
	    				result.category_list.map((item, index) => {
	    					item.value = index;
	    					item.label = item.reader_id;
	    				})
	    				this.recommendForm.categoryList = result.category_list;
	    			}else{
	    				console.log(result)
	    			}
    			}catch(err){
    				console.log(err)
    			}
    		},
		    addCategoryFun(){
		    	this.showAddCategory = !this.showAddCategory;
		    },
		    submitrecommendForm(recommendForm) {
				this.$refs[recommendForm].validate(async (valid) => {
					if (valid) {
						const params = {
							reader_id: this.recommendForm.reader_id,
							description: this.recommendForm.description,
							borrow_id: this.borrow_id,
						}
						try{
							const result = await addCategory(params);
							if (result.status == 1) {
								this.initData();
								this.recommendForm.reader_id = '';
								this.recommendForm.description = '';
								this.showAddCategory = false;
								this.$message({
					            	type: 'success',
					            	message: '添加成功'
					          	});
							}
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
			handleDelete(index){
				this.returnForm.specs.splice(index, 1);
			},
			tableRowClassName(row, index) {
		        if (index === 1) {
		        	return 'info-row';
		        } else if (index === 3) {
		        	return 'positive-row';
		        }
		        return '';
		    },
		    returns(returnForm){
		    	this.$refs[returnForm].validate(async (valid) => {
					if (valid) {
						const params = {
							...this.returnForm,
							category_id: this.selectValue.id,
							borrow_id: this.borrow_id,
						}
						try{
							const result = await returns(params);
							if (result.status == 1) {
								console.log(result)
								this.$message({
					            	type: 'success',
					            	message: '添加成功'
					          	});
					          	this.returnForm = {
				    				book_id: '',
				    				description: '',
				    				time_slot_id: '',
				    			}
							}else{
								this.$message({
					            	type: 'error',
					            	message: result.message
					          	});
							}
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
		    }
		}
    }</script>

<style lang="less">
	@import '../style/mixin';
	.form{
		min-width: 400px;
		margin-bottom: 30px;
		&:hover{
			box-shadow: 0 0 8px 0 rgba(232,237,250,.6), 0 2px 4px 0 rgba(232,237,250,.5);
			border-radius: 6px;
			transition: all 400ms;
		}
	}
	.return_form{
		border: 1px solid #eaeefb;
		padding: 10px 10px 0;
	}
	.form_header{
		text-align: center;
		margin-bottom: 10px;
	}
	.category_select{
		border: 1px solid #eaeefb;
		padding: 10px 30px 10px 10px;
		border-top-right-radius: 6px;
		border-top-left-radius: 6px;
	}
	.add_category_row{
		height: 0;
		overflow: hidden;
		transition: all 400ms;
		background: #f9fafc;
	}
	.showEdit{
		height: 185px;
	}
	.add_category{
		background: #f9fafc;
		padding: 10px 30px 0px 10px;
		border: 1px solid #eaeefb;
		border-top: none;
	}
	.add_category_button{
		text-align: center;
		line-height: 40px;
		border-bottom-right-radius: 6px;
		border-bottom-left-radius: 6px;
		border: 1px solid #eaeefb;
		border-top: none;
		transition: all 400ms;
		&:hover{
			background: #f9fafc;
			span, .edit_icon{
				color: #20a0ff;
			}
		}
		span{
			.sc(14px, #999);
			transition: all 400ms;
		}
		.edit_icon{
			color: #ccc;
			transition: all 400ms;
		}
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
	.cell{
		text-align: center;
	}
</style>
