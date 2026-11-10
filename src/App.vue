<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4 md:p-8">
    <div class="max-w-7xl mx-auto">
      <!-- 标题 -->
      <header class="text-center mb-8">
        <h1 class="text-4xl md:text-5xl font-bold text-gray-800 mb-2">
          🎨 在线印章生成器
        </h1>
        <p class="text-gray-600">轻松制作个性化电子印章</p>
      </header>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <!-- 左侧控制面板 -->
        <div class="bg-white rounded-xl shadow-lg p-6 space-y-6">
          <h2 class="text-2xl font-semibold text-gray-800 border-b pb-3">
            ⚙️ 印章设置
          </h2>

          <!-- 印章文字 -->
          <div class="space-y-3">
            <label class="block text-sm font-medium text-gray-700">
              单位名称（环形文字）
            </label>
            <input
              v-model="sealConfig.text"
              type="text"
              placeholder="请输入单位法定名称"
              class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            />
            <p class="text-xs text-gray-500">将沿印章上半圆从左到右排列</p>
          </div>

          <!-- 印章编码 -->
          <div class="space-y-3">
            <label class="block text-sm font-medium text-gray-700">印章编码（可选）</label>
            <input
              v-model="sealConfig.centerText"
              type="text"
              placeholder="如：1101010123456 或 ABC123"
              class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            />
            <p class="text-xs text-gray-500">将横排在印章最底部</p>
          </div>

          <!-- 底部文字 -->
          <div class="space-y-3">
            <label class="block text-sm font-medium text-gray-700">印章类型（横排文字）</label>
            <input
              v-model="sealConfig.bottomText"
              type="text"
              placeholder="如：专用章、合同专用章"
              class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            />
            <p class="text-xs text-gray-500">将横排在五角星下方</p>
          </div>

          <!-- 字体大小 -->
          <div class="space-y-3">
            <label class="block text-sm font-medium text-gray-700">
              字体大小: {{ sealConfig.fontSize }}px
            </label>
            <input
              v-model.number="sealConfig.fontSize"
              type="range"
              min="16"
              max="36"
              class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
            />
            <p class="text-xs text-gray-500">建议使用宋体，标准公章文字高度约6.5-7mm</p>
          </div>

          <!-- 环形文字角度范围 -->
          <div class="space-y-3">
            <label class="block text-sm font-medium text-gray-700">
              环形文字展开角度: {{ sealConfig.textAngle }}°
            </label>
            <input
              v-model.number="sealConfig.textAngle"
              type="range"
              min="120"
              max="270"
              step="5"
              class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
            />
            <p class="text-xs text-gray-500">数值越小文字越紧凑</p>
          </div>

          <!-- 印章尺寸 -->
          <div class="space-y-3">
            <label class="block text-sm font-medium text-gray-700">
              印章直径: {{ sealConfig.size }}px
            </label>
            <input
              v-model.number="sealConfig.size"
              type="range"
              min="200"
              max="450"
              step="10"
              class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
            />
            <p class="text-xs text-gray-500">标准公章直径：38mm、40mm、42mm、45mm</p>
          </div>

          <!-- 边框宽度 -->
          <div class="space-y-3">
            <label class="block text-sm font-medium text-gray-700">
              边框宽度: {{ sealConfig.borderWidth }}px
            </label>
            <input
              v-model.number="sealConfig.borderWidth"
              type="range"
              min="2"
              max="8"
              class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
            />
            <p class="text-xs text-gray-500">标准边框宽度：1mm-1.5mm</p>
          </div>

          <!-- 操作按钮 -->
          <div class="flex gap-3 pt-4">
            <button
              @click="resetConfig"
              class="flex-1 px-6 py-3 border-2 border-gray-300 hover:border-gray-400 text-gray-700 font-semibold rounded-lg transition-colors"
            >
              重置
            </button>
          </div>
        </div>

        <!-- 右侧预览区域 -->
        <div class="bg-white rounded-xl shadow-lg p-6">
          <h2 class="text-2xl font-semibold text-gray-800 border-b pb-3 mb-6">
            👁️ 印章预览
          </h2>
          
          <div class="flex items-center justify-center min-h-[400px] bg-gray-50 rounded-lg">
            <canvas
              ref="canvasRef"
              :width="sealConfig.size"
              :height="sealConfig.size"
              class="max-w-full"
              :style="{ transform: `rotate(${sealConfig.rotation}deg)` }"
            />
          </div>

          <!-- 旋转角度控制 -->
          <div class="mt-6 space-y-3">
            <label class="block text-sm font-medium text-gray-700">
              整体旋转角度: {{ sealConfig.rotation }}°
            </label>
            <input
              v-model.number="sealConfig.rotation"
              type="range"
              min="-180"
              max="180"
              step="1"
              class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
            />
            <p class="text-xs text-gray-500">可微调印章角度，模拟真实盖章效果</p>
          </div>

          <!-- 下载按钮 -->
          <div class="mt-6">
            <button
              @click="downloadSeal"
              class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-6 rounded-lg transition-colors flex items-center justify-center gap-2"
            >
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path>
              </svg>
              下载印章
            </button>
          </div>

          <div class="mt-4 text-center text-sm text-gray-500">
            <p>提示：点击"下载印章"按钮可保存为PNG图片</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, watch, onMounted } from 'vue'

const canvasRef = ref(null)

// 印章配置
const sealConfig = reactive({
  text: '厦门市集美区华丽小吃店（个体工商户）',
  centerText: '91350211MAK9TJYN6G', // 用于印章编码（13位数字）
  bottomText: '合同专用章',
  fontSize: 24,
  size: 300,
  borderWidth: 4,
  textAngle: 250, // 环形文字展开角度，默认200度
  rotation: 20 // 整体旋转角度，默认0度
})

// 颜色选项（固定为红色）
const sealColor = '#DC2626'

// 绘制圆形印章（按照中国公章标准）
const drawRoundSeal = (ctx, centerX, centerY, radius) => {
  const { text, centerText, bottomText, fontSize, borderWidth, textAngle } = sealConfig
  const color = sealColor
  
  // 清空画布
  ctx.clearRect(0, 0, sealConfig.size, sealConfig.size)
  
  // 绘制外圆边框
  ctx.beginPath()
  ctx.arc(centerX, centerY, radius, 0, Math.PI * 2)
  ctx.strokeStyle = color
  ctx.lineWidth = borderWidth
  ctx.stroke()
  
  // 绘制环形文字（单位法定名称）- 从左到右顺时针排列，以顶部为中心对称
  if (text) {
    const chars = text.split('')
    // 使用用户设置的角度值
    const totalAngle = (textAngle * Math.PI) / 180 // 转换为弧度
    
    ctx.font = `bold ${fontSize}px "SimSun", "STSong", "宋体", serif`
    
    // 计算文字半径
    const textRadius = radius - borderWidth - fontSize * 0.8
    
    // 根据字符数量分配角度
    const anglePerChar = totalAngle / chars.length
    
    // 起始角度：以顶部（-90°或270°）为中心对称分布
    const startAngle = -Math.PI / 2 - totalAngle / 2 + anglePerChar / 2
    
    ctx.fillStyle = color
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    
    chars.forEach((char, index) => {
      const angle = startAngle + anglePerChar * index
      const x = centerX + textRadius * Math.cos(angle)
      const y = centerY + textRadius * Math.sin(angle)
      
      ctx.save()
      ctx.translate(x, y)
      // 旋转文字使其垂直于半径方向
      ctx.rotate(angle + Math.PI / 2)
      ctx.fillText(char, 0, 0)
      ctx.restore()
    })
  }
  
  // 绘制中心五角星
  drawFivePointedStar(ctx, centerX, centerY, radius * 0.25, color)
  
  // 绘制底部横排文字（如"专用章"）
  if (bottomText) {
    ctx.font = `bold ${fontSize * 0.9}px "SimSun", "STSong", "宋体", serif`
    ctx.fillStyle = color
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    // 底部文字位置：五角星下方
    ctx.fillText(bottomText, centerX, centerY + radius * 0.35)
  }
  
  // 绘制印章编码（沿底部内侧反弧排列，居中对齐）
  if (centerText) {
    const codeChars = centerText.split('')
    // 底部反弧角度范围：更紧凑，每个字符最小5度
    const minAnglePerChar = (5 * Math.PI) / 180 // 每个字符最小5度，更紧凑
    const codeTotalAngle = Math.min((100 * Math.PI) / 180, minAnglePerChar * codeChars.length)
    const codeAngleStep = codeTotalAngle / Math.max(codeChars.length, 1)
    
    // 起始角度：以正下方（90°）为中心对称分布
    // 从左侧开始：90° - 总角度/2
    const codeStartAngle = Math.PI / 2 - codeTotalAngle / 2 + codeAngleStep / 2
    
    ctx.font = `${fontSize * 0.5}px Arial, sans-serif`
    ctx.fillStyle = color
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    
    codeChars.forEach((char, index) => {
      // 反转索引，让编码从左到右正确显示
      const reverseIndex = codeChars.length - 1 - index
      const angle = codeStartAngle + codeAngleStep * reverseIndex
      // 编码位置：靠近底部边缘内侧
      const codeRadius = radius - borderWidth - fontSize * 0.6
      const x = centerX + codeRadius * Math.cos(angle)
      const y = centerY + codeRadius * Math.sin(angle)
      
      ctx.save()
      ctx.translate(x, y)
      // 旋转文字使其垂直于半径方向（反弧向上）
      ctx.rotate(angle - Math.PI / 2)
      ctx.fillText(char, 0, 0)
      ctx.restore()
    })
  }
}

// 绘制五角星
const drawFivePointedStar = (ctx, centerX, centerY, radius, color) => {
  ctx.save()
  ctx.beginPath()
  ctx.fillStyle = color
  
  for (let i = 0; i < 5; i++) {
    const outerAngle = (i * 4 * Math.PI) / 5 - Math.PI / 2
    const innerAngle = ((i * 4 + 2) * Math.PI) / 5 - Math.PI / 2
    
    const outerX = centerX + radius * Math.cos(outerAngle)
    const outerY = centerY + radius * Math.sin(outerAngle)
    const innerX = centerX + radius * 0.38 * Math.cos(innerAngle)
    const innerY = centerY + radius * 0.38 * Math.sin(innerAngle)
    
    if (i === 0) {
      ctx.moveTo(outerX, outerY)
    } else {
      ctx.lineTo(outerX, outerY)
    }
    ctx.lineTo(innerX, innerY)
  }
  
  ctx.closePath()
  ctx.fill()
  ctx.restore()
}

// 绘制印章
const drawSeal = () => {
  if (!canvasRef.value) return
  
  const canvas = canvasRef.value
  const ctx = canvas.getContext('2d')
  const centerX = sealConfig.size / 2
  const centerY = sealConfig.size / 2
  
  // 仅绘制圆形印章
  drawRoundSeal(ctx, centerX, centerY, sealConfig.size / 2 - 10)
}

// 下载印章
const downloadSeal = () => {
  if (!canvasRef.value) return
  
  const canvas = canvasRef.value
  const ctx = canvas.getContext('2d')
  
  // 如果有旋转角度，需要创建临时Canvas进行旋转绘制
  if (sealConfig.rotation !== 0) {
    // 创建临时Canvas
    const tempCanvas = document.createElement('canvas')
    const size = sealConfig.size
    tempCanvas.width = size
    tempCanvas.height = size
    const tempCtx = tempCanvas.getContext('2d')
    
    // 保存当前状态
    tempCtx.save()
    
    // 移动到中心点并旋转
    tempCtx.translate(size / 2, size / 2)
    tempCtx.rotate((sealConfig.rotation * Math.PI) / 180)
    tempCtx.translate(-size / 2, -size / 2)
    
    // 绘制原始印章（不旋转）
    const originalRotation = sealConfig.rotation
    sealConfig.rotation = 0
    drawRoundSeal(tempCtx, size / 2, size / 2, size / 2 - 10)
    sealConfig.rotation = originalRotation
    
    // 恢复状态
    tempCtx.restore()
    
    // 下载旋转后的图片
    const link = document.createElement('a')
    link.download = `印章_${Date.now()}.png`
    link.href = tempCanvas.toDataURL('image/png')
    link.click()
  } else {
    // 没有旋转，直接下载
    const link = document.createElement('a')
    link.download = `印章_${Date.now()}.png`
    link.href = canvas.toDataURL('image/png')
    link.click()
  }
}

// 重置配置
const resetConfig = () => {
  sealConfig.text = '示例有限公司'
  sealConfig.centerText = ''
  sealConfig.bottomText = '专用章'
  sealConfig.fontSize = 24
  sealConfig.size = 300
  sealConfig.borderWidth = 4
  sealConfig.textAngle = 200
  sealConfig.rotation = 0
}

// 监听配置变化，重新绘制
watch(sealConfig, () => {
  // 使用nextTick确保DOM更新后再绘制
  setTimeout(() => {
    drawSeal()
  }, 0)
}, { deep: true })

// 初始化绘制
onMounted(() => {
  drawSeal()
})
</script>

<style scoped>
/* 自定义滚动条样式 */
input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 16px;
  height: 16px;
  background: #3B82F6;
  border-radius: 50%;
  cursor: pointer;
}

input[type="range"]::-moz-range-thumb {
  width: 16px;
  height: 16px;
  background: #3B82F6;
  border-radius: 50%;
  cursor: pointer;
  border: none;
}
</style>
