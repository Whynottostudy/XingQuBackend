<!-- PTR-FR-001 -->


<template>
  <view class="container">
    <view class="form-container">
      <!-- 旅游计划输入区域 -->
      <view class="form-title">请填写您的旅游计划</view>

      <!-- 目的地 -->
      <view class="form-item">
        <text class="label">目的地：</text>
        <input type="text" placeholder="如福建省福州市鼓楼区，仅限国内" v-model="formData.destination" />
      </view>

      <!-- 时间选择 -->
      <view class="form-item">
        <text class="label">时间：</text>
        <view class="date-picker">
          <input type="text" placeholder="如2024/10/25" v-model="formData.startDate" />
          <text class="separator">至</text>
          <input type="text" placeholder="如2024/11/04" v-model="formData.endDate" />
        </view>
      </view>

      <!-- 预算 -->
      <view class="form-item">
        <text class="label">预算：</text>
        <view class="budget-input">
          <input type="number" placeholder="请输入数字" v-model="formData.budgetMin" />
          <text class="separator">—</text>
          <input type="number" placeholder="请输入数字" v-model="formData.budgetMax" />
          <text>元</text>
        </view>
      </view>

      <!-- 特殊需求选项 -->
      <view class="form-title sub">请勾选您对旅游景点的要求</view>
      
      <view class="checkbox-group">
        <view class="checkbox-item">
          <text>无障碍设施：</text>
          <radio-group v-model="formData.accessibility">
            <radio value="yes">是</radio>
            <radio value="no">否</radio>
          </radio-group>
        </view>

        <view class="checkbox-item">
          <text>亲子友好：</text>
          <radio-group v-model="formData.kidFriendly">
            <radio value="yes">是</radio>
            <radio value="no">否</radio>
          </radio-group>
        </view>

        <view class="checkbox-item">
          <text>宠物友好：</text>
          <radio-group v-model="formData.petFriendly">
            <radio value="yes">是</radio>
            <radio value="no">否</radio>
          </radio-group>
        </view>
      </view>

      <!-- 个性需求输入区域 -->
      <view class="form-title sub">请填写您对旅游景点的个性需求(选填)</view>
      <view class="requirement-section">
        <view class="form-item">
          <text class="required">*</text>
          <text>自然景观：</text>
          <input type="text" placeholder="如草原/海滩/雪山" v-model="formData.naturalLandscape" />
        </view>

        <view class="form-item">
          <text class="required">*</text>
          <text>社会景观：</text>
          <input type="text" placeholder="如纪念馆/博物馆/科技馆" v-model="formData.socialLandscape" />
        </view>

        <view class="form-item">
          <text class="required">*</text>
          <text>周边饮食：</text>
          <input type="text" placeholder="如闽菜/川菜/粤菜" v-model="formData.cuisine" />
        </view>

        <view class="form-item">
          <text class="required">*</text>
          <text>其余需求：</text>
          <input type="text" placeholder="填写对景点的其余需求" v-model="formData.otherNeeds" />
        </view>
      </view>

      <!-- 提交按钮 -->
      <button class="submit-btn" @click="submitForm">
        提交获得景点推荐
        
      </button>
    </view>
  </view>
</template>

<script lang="ts">
import { defineComponent, reactive } from 'vue'

export default defineComponent({
  setup() {
    const formData = reactive({
      destination: '',
      startDate: '',
      endDate: '',
      budgetMin: '',
      budgetMax: '',
      accessibility: 'no',
      kidFriendly: 'no',
      petFriendly: 'no',
      naturalLandscape: '',
      socialLandscape: '',
      cuisine: '',
      otherNeeds: ''
    })

    const submitForm = async () => {
      // 表单验证（可以根据需要进行具体的表单验证）
		const budgetMin = parseFloat(formData.budgetMin);
		const budgetMax = parseFloat(formData.budgetMax);
		if (isNaN(budgetMin)) {
		    formData.budgetMin = 0;
		  } else {
		    formData.budgetMin = budgetMin;
		  }
		
		  if (isNaN(budgetMax)) {
		    formData.budgetMax = 0;
		  } else {
		    formData.budgetMax = budgetMax;
		  }
		  
      try {
        const db = uniCloud.database();
        const collection = db.collection('travel_plan'); // 假设你创建了一个集合名为 travel_plan
    
        // 提交数据到云数据库
        await collection.add({
          destination: formData.destination,
          startDate: formData.startDate,
          endDate: formData.endDate,
          budgetMin: formData.budgetMin,
          budgetMax: formData.budgetMax,
          accessibility: formData.accessibility,
          kidFriendly: formData.kidFriendly,
          petFriendly: formData.petFriendly,
          naturalLandscape: formData.naturalLandscape,
          socialLandscape: formData.socialLandscape,
          cuisine: formData.cuisine,
          otherNeeds: formData.otherNeeds,
          // 添加其他可能的字段
          createTime: new Date(), // 保存提交时间
        });
    
        // 提交成功后跳转到推荐结果页面
        uni.navigateTo({
          url: '/pages/tuijianjieguo/tuijianjieguo',
          success: (res) => {
            // 通过 eventChannel 向被打开页面传送数据
            res.eventChannel.emit('acceptFormData', formData);
          },
        });
      } catch (error) {
        console.error('提交失败', error);
        uni.showToast({
          title: '提交失败，请重试',
          icon: 'none',
        });
      }
    };

    return {
      formData,
      submitForm
    }
  }
})
</script>

<style scoped>
.container {
  padding: 20px;
  background-color: #f5f5f5;
}

.form-container {
  background-color: #fff;
  border-radius: 10px;
  padding: 20px;
}

.form-title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 20px;
}

.form-title.sub {
  margin-top: 20px;
}

.form-item {
  margin-bottom: 15px;
  display: flex;
  align-items: center;
}

.label {
  width: 80px;
  flex-shrink: 0;
}

input {
  flex: 1;
  border: 1px solid #ddd;
  padding: 8px;
  border-radius: 4px;
}

.date-picker {
  display: flex;
  align-items: center;
  gap: 10px;
}

.budget-input {
  display: flex;
  align-items: center;
  gap: 10px;
}

.separator {
  color: #666;
}

.checkbox-group {
  margin: 15px 0;
}

.checkbox-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

radio-group {
  display: flex;
  gap: 20px;
}

.required {
  color: #ff4d4f;
  margin-right: 5px;
}

.requirement-section {
  margin-top: 15px;
}

.submit-btn {
  margin-top: 10px;
  width: 100%;
  height:70px;
  background-color: #007AFF;
  color: white;
  border: none;
  padding: 10px;
  border-radius: 25px;
  display: flex;
  flex-direction: column;
  align-items: center;
  
}


</style> 
