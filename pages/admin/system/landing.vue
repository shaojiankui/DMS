<template>
  <div class="app-container">
    <!-- 页面头部 -->
    <!-- <div class="page-header">
      <h1 class="page-title">默认着陆页配置</h1>
      <p class="page-description">配置所有域名着陆页的默认样式和功能选项</p>
    </div> -->
    <div class="action-buttons">
      <el-button
        type="primary"
        @click="saveConfig"
        :loading="saving"
        size="mini"
        plain
      >
        <i class="el-icon-check"></i>
        保存
      </el-button>
      <el-button @click="resetConfig" size="mini" plain>
        <i class="el-icon-refresh"></i>
        重置
      </el-button>
    </div>
    <div class="config-layout">
      
      <!-- 左侧配置面板 -->
      <div class="config-panel">
        <el-form :model="configForm" label-width="30px" ref="configFormRef">
          <h3 class="section-title">米表主题</h3>
          <el-form-item label="" prop="colorTheme">
              <el-select v-model="configForm.backgroundColor" placeholder="选择米表主题风格" style="width: 100%" @change="selectBackgroundColor">
                <el-option
                  v-for="theme in backgroundColors"
                  :key="theme.value"
                  :label="theme.label"
                  :value="theme.value"
                >
                  <div style="display: flex; justify-content: space-between; align-items: center;">
                    <span>{{ theme.label }}</span>
                    <span style="color: #999; font-size: 12px;">{{ theme.description }}</span>
                  </div>
                </el-option>
              </el-select>
            </el-form-item>
          <!-- 出售选项 -->
          <div class="config-section">
            <h3 class="section-title">出售选项</h3>
            <el-form-item prop="saleOptions">
              <el-checkbox-group
                v-model="configForm.saleOptions"
                @change="updatePreview"
              >
                <div class="sale-options">
                  <el-checkbox label="directBuy" class="sale-option">
                    <div class="option-content">
                      <span class="option-label">直接购买</span>
                      <span class="option-desc"
                        >显示固定价格，支持立即购买</span
                      >
                    </div>
                  </el-checkbox>
                  <el-checkbox label="makeOffer" class="sale-option">
                    <div class="option-content">
                      <span class="option-label">提出报价</span>
                      <span class="option-desc">允许访客提交价格报价</span>
                    </div>
                  </el-checkbox>
                  <el-checkbox label="inquiry" class="sale-option">
                    <div class="option-content">
                      <span class="option-label">询价表单</span>
                      <span class="option-desc">显示询价联系表单</span>
                    </div>
                  </el-checkbox>
                </div>
              </el-checkbox-group>
            </el-form-item>
          </div>

          <!-- 标题配置 -->
          <div class="config-section">
            <h3 class="section-title">标题配置</h3>
            <p class="section-desc">设置着陆页显示的标题文本</p>
            <el-form-item
              label="主标题"
              prop="mainTitle"
              label-width="100px"
            >
              <el-input
                v-model="configForm.mainTitle"
                placeholder="请输入主标题"
                @input="updatePreview"
              />
            </el-form-item>
            <el-form-item
              label="副标题"
              prop="subTitle"
              label-width="100px"
            >
              <el-input
                v-model="configForm.subTitle"
                placeholder="请输入副标题"
                @input="updatePreview"
              />
            </el-form-item>
            <el-form-item
              label="底部标题"
              prop="footerTitle"
              label-width="100px"
            >
              <el-input
                v-model="configForm.footerTitle"
                placeholder="请输入底部标题"
                @input="updatePreview"
              />
            </el-form-item>
            <el-form-item
              label="版权信息"
              prop="copyrightTitle"
              label-width="100px"
            >
              <el-input
                v-model="configForm.copyrightTitle"
                placeholder="请输入版权信息"
                @input="updatePreview"
              />
            </el-form-item>
          </div>

          <!-- 流量图表 -->
          <div class="config-section">
            <h3 class="section-title">流量图表</h3>
            <p class="section-desc">
              为购买者展示您域名的流量数据，向客户展示价值
            </p>
            <el-form-item prop="trafficChart">
              <el-radio-group
                v-model="configForm.trafficChart"
                @change="updatePreview"
              >
                <el-radio label="always">始终查看</el-radio>
                <el-radio label="monthly100">仅限月访问量100+</el-radio>
                <el-radio label="never">始终关闭</el-radio>
              </el-radio-group>
            </el-form-item>
          </div>

          <!-- 联系信息 -->
          <div class="config-section">
            <h3 class="section-title">联系信息</h3>
            <el-form-item
              label="联系邮箱"
              prop="contactEmail"
              label-width="100px"
            >
              <el-input
                v-model="configForm.contactEmail"
                placeholder="请输入联系邮箱"
                @input="updatePreview"
              />
            </el-form-item>
            <el-form-item
              label="联系电话"
              prop="contactPhone"
              label-width="100px"
            >
              <el-input
                v-model="configForm.contactPhone"
                placeholder="请输入联系电话"
                @input="updatePreview"
              />
            </el-form-item>
            <el-form-item
              label="公司名称"
              prop="companyName"
              label-width="100px"
            >
              <el-input
                v-model="configForm.companyName"
                placeholder="请输入公司名称"
                @input="updatePreview"
              />
            </el-form-item>
          </div>

          <!-- 自定义CSS -->
          <div class="config-section">
            <h3 class="section-title">自定义样式</h3>
            <p class="section-desc">添加自定义CSS来个性化您的着陆页</p>
            <el-form-item prop="customCSS">
              <el-input
                v-model="configForm.customCSS"
                type="textarea"
                :rows="6"
                placeholder="请输入自定义CSS代码"
                @input="updatePreview"
              />
            </el-form-item>
          </div>
        </el-form>
       
      </div>

      <!-- 右侧预览面板 -->
      <div class="preview-panel">
        <div class="preview-header">
          <h3>预览效果</h3>
          <div class="preview-controls">
            <el-button-group>
              <el-button
                :type="previewDevice === 'desktop' ? 'primary' : ''"
                @click="previewDevice = 'desktop'"
                size="small"
              >
                桌面端
              </el-button>
              <el-button
                :type="previewDevice === 'mobile' ? 'primary' : ''"
                @click="previewDevice = 'mobile'"
                size="small"
              >
                移动端
              </el-button>
            </el-button-group>
            <el-button size="small" @click="refreshPreview">刷新</el-button>
          </div>
        </div>
        <div class="preview-content" :class="previewDevice">
          <div class="preview-frame" v-html="previewHTML"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ElMessage } from "element-plus";
import { 
  LAYOUT_TEMPLATES, 
  COLOR_THEMES, 
  getTemplateLabel,
  getThemeLabel
} from '~/utils/constants.js'
definePageMeta({
  layout: "admin",
  title: "默认着陆页配置",
  middleware: "auth",
});

useHead({
  title: "默认着陆页配置 - DMS 管理后台",
});

// refs
const configFormRef = ref();

// 响应式数据
const saving = ref(false);
const previewDevice = ref("desktop");
const previewHTML = ref("");
const backgroundColors = ref(COLOR_THEMES)


// 表单数据
const configForm = reactive({
  theme: "standard",
  backgroundType: "solid",
  backgroundColor: "lavender",
  customBackgroundColor: "#1a1a1a",
  fontColor: "auto",
  saleOptions: ["inquiry", "makeOffer"],
  priceDisplay: "hide",
  trafficChart: "never",
  storePromotion: "hide",
  contactEmail: "",
  contactPhone: "",
  companyName: "",
  customCSS: "",
  mainTitle: "域名正在出售",
  subTitle: "这个优质域名正在出售中",
  footerTitle: "© 2024 域名大师 - 专业域名交易平台",
  copyrightTitle: "保留所有权利",
});

// 选择背景颜色
function selectBackgroundColor(value: string) {
  configForm.backgroundColor = value;
  updatePreview();
}

// 更新预览
function updatePreview() {
  generatePreviewHTML();
}

// 生成预览HTML
function generatePreviewHTML() {
  const selectedBgColor = backgroundColors.value.find(
    (c) => c.value === configForm.backgroundColor
  );
  const bgStyle = `background: ${selectedBgColor?.primary}`
  console.log(bgStyle)

  const fontColorStyle =
    configForm.fontColor === "dark"
      ? "color: #333"
      : configForm.fontColor === "light"
      ? "color: #fff"
      : "color: #fff"; // auto默认为白色

  let actionsHTML = "";
  if (configForm.saleOptions.includes("directBuy")) {
    actionsHTML +=
      '<button class="action-btn primary">立即购买 ￥8,888</button>';
  }
  if (configForm.saleOptions.includes("makeOffer")) {
    actionsHTML += '<button class="action-btn secondary">提出报价</button>';
  }
  if (configForm.saleOptions.includes("inquiry")) {
    actionsHTML += '<button class="action-btn secondary">询价咨询</button>';
  }
  if (configForm.saleOptions.includes("auction")) {
    actionsHTML += '<button class="action-btn warning">参与竞价</button>';
  }

  let priceHTML = "";
  if (configForm.priceDisplay === "show") {
    priceHTML =
      '<div class="price-section"><div class="price">￥8,888</div><div class="price-label">售价</div></div>';
  } else if (configForm.priceDisplay === "onRequest") {
    priceHTML =
      '<div class="price-section"><div class="price">面议</div><div class="price-label">价格</div></div>';
  }

  const contactHTML =
    configForm.contactEmail || configForm.contactPhone || configForm.companyName
      ? `
    <div class="contact-section">
      <h3>联系我们</h3>
      ${configForm.companyName ? `<p>公司：${configForm.companyName}</p>` : ""}
      ${
        configForm.contactEmail ? `<p>邮箱：${configForm.contactEmail}</p>` : ""
      }
      ${
        configForm.contactPhone ? `<p>电话：${configForm.contactPhone}</p>` : ""
      }
    </div>
  `
      : "";

  previewHTML.value = `
    <div style="min-height: 100vh; ${bgStyle}; ${fontColorStyle}; font-family: Arial, sans-serif;">
      <style>
        .preview-container { max-width: 800px; margin: 0 auto; padding: 40px 20px; text-align: center; }
        .domain-title { font-size: 48px; font-weight: bold; margin-bottom: 20px; }
        .domain-subtitle { font-size: 18px; opacity: 0.8; margin-bottom: 40px; }
        .price-section { margin: 30px 0; }
        .price { font-size: 36px; font-weight: bold; margin-bottom: 5px; }
        .price-label { font-size: 14px; opacity: 0.7; }
        .actions { margin: 40px 0; display: flex; gap: 15px; justify-content: center; flex-wrap: wrap; }
        .action-btn { padding: 12px 24px; border: none; border-radius: 6px; font-size: 16px; cursor: pointer; transition: all 0.3s; }
        .action-btn.primary { background: #007bff; color: white; }
        .action-btn.secondary { background: rgba(255,255,255,0.2); color: inherit; border: 1px solid rgba(255,255,255,0.3); }
        .action-btn.warning { background: #ffc107; color: #000; }
        .action-btn:hover { transform: translateY(-2px); opacity: 0.9; }
        .contact-section { margin-top: 60px; padding: 30px; background: rgba(255,255,255,0.1); border-radius: 10px; }
        .contact-section h3 { margin-bottom: 20px; }
        .contact-section p { margin: 8px 0; }
        .features { margin: 50px 0; display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; }
        .feature { padding: 20px; background: rgba(255,255,255,0.1); border-radius: 8px; }
        .footer { margin-top: 60px; padding: 20px; font-size: 14px; opacity: 0.7; }
        .copyright-info { font-size: 12px; margin-top: 8px; opacity: 0.8; }
        ${configForm.customCSS}
      </style>
      <div class="preview-container">
        <h1 class="domain-title">example.com</h1>
        <p class="domain-subtitle">${configForm.subTitle}</p>
        
        ${priceHTML}
        
        <div class="actions">
          ${actionsHTML}
        </div>
        
        ${
          configForm.theme === "branded"
            ? `
          <div class="features">
            <div class="feature">
              <h4>优质域名</h4>
              <p>短小精悍，易于记忆</p>
            </div>
            <div class="feature">
              <h4>商业价值</h4>
              <p>适合各种商业用途</p>
            </div>
            <div class="feature">
              <h4>投资潜力</h4>
              <p>具有升值空间</p>
            </div>
          </div>
        `
            : ""
        }
        
        ${
          configForm.saleOptions.includes("inquiry")
            ? `
          <div class="inquiry-form" style="margin-top: 40px; padding: 30px; background: rgba(255,255,255,0.1); border-radius: 10px; max-width: 500px; margin-left: auto; margin-right: auto;">
            <h3>${configForm.mainTitle}</h3>
            <form style="display: grid; gap: 15px; text-align: left;">
              <input type="text" placeholder="您的姓名" style="padding: 10px; border: 1px solid rgba(255,255,255,0.3); background: rgba(255,255,255,0.1); border-radius: 4px; color: inherit;">
              <input type="email" placeholder="联系邮箱" style="padding: 10px; border: 1px solid rgba(255,255,255,0.3); background: rgba(255,255,255,0.1); border-radius: 4px; color: inherit;">
              <input type="text" placeholder="预期价格" style="padding: 10px; border: 1px solid rgba(255,255,255,0.3); background: rgba(255,255,255,0.1); border-radius: 4px; color: inherit;">
              <textarea placeholder="留言内容" rows="3" style="padding: 10px; border: 1px solid rgba(255,255,255,0.3); background: rgba(255,255,255,0.1); border-radius: 4px; color: inherit; resize: vertical;"></textarea>
              <button type="submit" style="padding: 12px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer;">提交询价</button>
            </form>
          </div>
        `
            : ""
        }
        
        ${contactHTML}
        
        <div class="footer">
          ${configForm.footerTitle}
          ${configForm.copyrightTitle ? `<div class="copyright-info">${configForm.copyrightTitle}</div>` : ''}
        </div>
      </div>
    </div>
  `;
}

// 保存配置
async function saveConfig() {
  saving.value = true;
  try {
    const response = (await $fetch("/api/admin/system/landing-config/save", {
      method: "POST",
      body: configForm,
    })) as any;

    if (response.code === 200) {
      ElMessage.success("配置保存成功");
    } else {
      ElMessage.error(response.message || "配置保存失败");
    }
  } catch (error) {
    console.error("保存配置失败:", error);
    ElMessage.error("保存配置失败");
  } finally {
    saving.value = false;
  }
}

// 重置配置
function resetConfig() {
  Object.assign(configForm, {
    theme: "standard",
    backgroundType: "solid",
    backgroundColor: "dark",
    customBackgroundColor: "#1a1a1a",
    fontColor: "auto",
    saleOptions: ["inquiry", "makeOffer"],
    priceDisplay: "hide",
    trafficChart: "never",
    storePromotion: "hide",
    contactEmail: "",
    contactPhone: "",
    companyName: "",
    customCSS: "",
    mainTitle: "域名正在出售",
    subTitle: "这个优质域名正在出售中",
    footerTitle: "© 2024 域名大师 - 专业域名交易平台",
    copyrightTitle: "保留所有权利",
  });
  updatePreview();
  ElMessage.success("已重置为默认配置");
}

// 导出配置
function exportConfig() {
  const configData = JSON.stringify(configForm, null, 2);
  const blob = new Blob([configData], { type: "application/json" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "landing-page-config.json";
  a.click();
  URL.revokeObjectURL(url);
  ElMessage.success("配置已导出");
}

// 刷新预览
function refreshPreview() {
  updatePreview();
}

// 加载已保存的配置
async function loadConfig() {
  try {
    const response = (await $fetch(
      "/api/admin/system/landing-config/get"
    )) as any;
    if (response.code === 200 && response.data) {
      Object.assign(configForm, response.data);
    }
  } catch (error) {
    console.error("加载配置失败:", error);
  }
  updatePreview();
}

// 页面加载时初始化
onMounted(() => {
  loadConfig();
});
</script>

<style scoped>
.app-container {
  padding: 15px;
  max-width: 1200px;
  margin: 0 auto;
}

.page-header {
  margin-bottom: 30px;
}

.page-title {
  font-size: 24px;
  font-weight: 600;
  margin: 0 0 8px 0;
  color: #1f2937;
}

.page-description {
  color: #6b7280;
  margin: 0;
}

.config-layout {
  display: flex;
  gap: 20px;
  height: calc(100vh - 160px);
}

.config-panel {
  flex: 1.5;
  overflow-y: auto;
  padding-right: 15px;
  padding-bottom: 100px; /* 为悬浮按钮留出空间 */
  position: relative; /* 为悬浮按钮提供定位基准 */
  height: 100%;
}

.config-section {
  margin-bottom: 25px;
  padding-bottom: 20px;
  border-bottom: 1px solid #e5e7eb;
}

.config-section:last-child {
  border-bottom: none;
}

.section-title {
  font-size: 18px;
  font-weight: 600;
  margin: 0 0 8px 0;
  color: #374151;
}

.section-desc {
  color: #6b7280;
  margin: 0 0 15px 0;
  font-size: 13px;
}

.theme-option {
  margin-left: 10px;
}

.theme-label {
  font-weight: 600;
  color: #374151;
}

.theme-desc {
  font-size: 12px;
  color: #6b7280;
  margin-top: 2px;
}

.color-options {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.color-option {
  width: 70px;
  height: 50px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border: 3px solid transparent;
  transition: all 0.3s;
  position: relative;
}

.color-option.active {
  border-color: #007bff;
  transform: scale(1.05);
}

.color-option.custom {
  background: linear-gradient(
      45deg,
      #f0f0f0 25%,
      transparent 25%,
      transparent 75%,
      #f0f0f0 75%
    ),
    linear-gradient(
      45deg,
      #f0f0f0 25%,
      transparent 25%,
      transparent 75%,
      #f0f0f0 75%
    );
  background-size: 10px 10px;
  background-position: 0 0, 5px 5px;
}

.color-name {
  font-size: 12px;
  font-weight: 600;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
  text-align: center;
}

.color-option.custom .color-name {
  color: #666;
  text-shadow: none;
}

.font-option {
  display: flex;
  align-items: center;
  gap: 5px;
}

.font-label {
  font-weight: 600;
}

.font-desc {
  font-size: 12px;
  color: #6b7280;
}

.sale-options {
  display: grid;
  gap: 10px;
}

.sale-option {
  display: block !important;
  width: 100%;
}
.theme-radio {
  height: 80px !important;
}
.option-content {
  margin-left: 10px;
  display: flex;
  flex-direction: column;
}

.option-label {
  font-weight: 600;
  color: #374151;
}

.option-desc {
  font-size: 12px;
  color: #6b7280;
  margin-top: 2px;
}



.action-buttons {
  display: flex;
  gap: 12px;
  align-items: center;
  padding-bottom: 10px;
}

.preview-panel {
  flex: 1;
  padding-left: 15px;
  display: flex;
  flex-direction: column;
}

.preview-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.preview-header h3 {
  margin: 0;
  color: #374151;
  font-size: 18px;
  font-weight: 600;
}

.preview-controls {
  display: flex;
  gap: 10px;
}

.preview-content {
  flex: 1;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
  background: white;
}

.preview-content.mobile {
  max-width: 375px;
  margin: 0 auto;
}

.preview-frame {
  width: 100%;
  height: 100%;
  overflow-y: auto;
}

:deep(.el-radio) {
  margin-bottom: 10px;
}

:deep(.el-checkbox) {
  margin-bottom: 10px;
}

:deep(.el-form-item) {
  margin-bottom: 15px;
}

:deep(.el-form-item__label) {
  font-weight: 600;
  color: #374151;
}

/* 响应式布局 */
@media (max-width: 1200px) {
  .config-layout {
    flex-direction: column;
    height: auto;
  }

  .config-panel {
    flex: none;
    padding-right: 0;
    margin-bottom: 20px;
  }

  .preview-panel {
    flex: none;
    border-left: none;
    border-top: 1px solid #e5e7eb;
    padding-left: 0;
    padding-top: 20px;
    height: 500px;
  }
}

@media (max-width: 768px) {
  .app-container {
    padding: 10px;
  }

  .color-options {
    gap: 8px;
  }

  .color-option {
    width: 60px;
    height: 40px;
  }

  .form-actions {
    flex-direction: column;
    gap: 8px;
  }


  .action-buttons {
    flex-direction: column;
    gap: 8px;
  }

  .action-buttons .el-button {
    width: 100%;
  }
}
</style>
