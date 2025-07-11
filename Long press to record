<template>
  <!-- 主容器 -->
  <div class="voice-recorder">
    <!-- 录音按钮 -->
    <button
      class="record-button"
      :class="{ 'recording': isRecording }"  <!-- 录音时添加recording类 -->
      @mousedown="handleStart"             <!-- 鼠标按下开始录音 -->
      @mouseup="handleStop"                <!-- 鼠标释放停止录音 -->
      @mouseleave="handleStop"             <!-- 鼠标离开按钮区域停止录音 -->
      @touchstart="handleStart"            <!-- 触摸开始(移动端支持) -->
      @touchend="handleStop"               <!-- 触摸结束(移动端支持) -->
    >
      {{ buttonText }}  <!-- 动态按钮文本 -->
    </button>
    
    <!-- 录音指示器 -->
    <div v-if="isRecording" class="recording-indicator">
      <div class="pulse"></div>  <!-- 脉冲动画效果 -->
      <span>录音中... {{ formattedTime }}</span>  <!-- 显示录音时间 -->
    </div>
    
    <!-- 录音回放区域 -->
    <div v-if="audioUrl" class="playback">
      <audio controls :src="audioUrl"></audio>  <!-- 音频播放控件 -->
      <button @click="downloadAudio" class="download-button">下载录音</button>
    </div>
    
    <!-- 错误信息显示 -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  setup() {
    // 响应式数据定义
    const isRecording = ref(false);         // 是否正在录音
    const mediaRecorder = ref(null);        // MediaRecorder实例引用
    const audioChunks = ref([]);            // 存储录音数据块
    const audioUrl = ref('');               // 录音完成后的URL
    const recordingTime = ref(0);           // 录音时长(秒)
    const error = ref('');                  // 错误信息
    let timer = null;                       // 录音计时器
    let audioContext = null;                // Web Audio API上下文
    let analyser = null;                    // 音频分析器
    let startTimeout = null;                // 长按开始计时器

    // 计算属性：格式化显示时间(MM:SS)
    const formattedTime = computed(() => {
      const minutes = Math.floor(recordingTime.value / 60);
      const seconds = recordingTime.value % 60;
      return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
    });

    // 计算属性：动态按钮文本
    const buttonText = computed(() => {
      return isRecording.value ? '松开结束' : '长按录音';
    });

    // 处理开始录音(长按逻辑)
    const handleStart = () => {
      error.value = '';  // 清除之前的错误
      // 设置300ms延迟，实现长按效果
      startTimeout = setTimeout(() => {
        startRecording();
      }, 300);
    };

    // 处理停止录音
    const handleStop = () => {
      // 清除长按计时器
      clearTimeout(startTimeout);
      // 如果正在录音，则停止
      if (isRecording.value) {
        stopRecording();
      }
    };

    // 开始录音的核心逻辑
    const startRecording = async () => {
      try {
        // 防止重复开始
        if (isRecording.value) return;
        
        // 重置状态
        audioChunks.value = [];
        audioUrl.value = '';
        recordingTime.value = 0;
        
        // 请求麦克风权限并获取音频流
        const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
        
        // 初始化Web Audio API相关对象
        audioContext = new AudioContext();
        analyser = audioContext.createAnalyser();
        const microphone = audioContext.createMediaStreamSource(stream);
        microphone.connect(analyser);  // 连接分析器，可用于后续音量可视化
        
        // 创建MediaRecorder实例，指定音频格式为webm
        mediaRecorder.value = new MediaRecorder(stream, { mimeType: 'audio/webm' });
        
        // 当有音频数据可用时的回调
        mediaRecorder.value.ondataavailable = (event) => {
          if (event.data.size > 0) {
            audioChunks.value.push(event.data);  // 存储音频数据块
          }
        };
        
        // 录音停止时的回调
        mediaRecorder.value.onstop = () => {
          // 将所有音频数据块合并为一个Blob对象
          const audioBlob = new Blob(audioChunks.value, { type: 'audio/webm' });
          // 创建可用于audio元素的URL
          audioUrl.value = URL.createObjectURL(audioBlob);
          // 停止所有音轨
          stream.getTracks().forEach(track => track.stop());
        };
        
        // 开始录音，每100ms收集一次数据
        mediaRecorder.value.start(100);
        isRecording.value = true;
        
        // 设置计时器，每秒更新录音时间
        timer = setInterval(() => {
          recordingTime.value += 1;
          // 这里可以添加音量检测逻辑
          // const dataArray = new Uint8Array(analyser.frequencyBinCount);
          // analyser.getByteFrequencyData(dataArray);
          // const volume = Math.max(...dataArray);
        }, 1000);
        
      } catch (err) {
        console.error('录音错误:', err);
        error.value = '无法访问麦克风，请检查权限设置';
        isRecording.value = false;
      }
    };

    // 停止录音的核心逻辑
    const stopRecording = () => {
      // 确保正在录音且有MediaRecorder实例
      if (isRecording.value && mediaRecorder.value) {
        clearInterval(timer);          // 清除计时器
        mediaRecorder.value.stop();    // 停止录音
        isRecording.value = false;     // 更新状态
      }
    };

    // 下载录音文件
    const downloadAudio = () => {
      if (!audioUrl.value) return;
      
      // 创建下载链接并触发点击
      const a = document.createElement('a');
      a.href = audioUrl.value;
      // 设置下载文件名，包含当前时间戳
      a.download = `recording-${new Date().toISOString()}.webm`;
      a.click();
    };

    // 暴露给模板的数据和方法
    return {
      isRecording,
      audioUrl,
      recordingTime,
      formattedTime,
      buttonText,
      error,
      handleStart,
      handleStop,
      downloadAudio
    };
  }
};
</script>

<style scoped>
/* 主容器样式 */
.voice-recorder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
}

/* 录音按钮基础样式 */
.record-button {
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  border: none;
  border-radius: 50px;
  background-color: #4CAF50; /* 绿色背景 */
  color: white;
  cursor: pointer;
  transition: all 0.2s;     /* 平滑过渡效果 */
  user-select: none;         /* 防止文字被选中 */
}

/* 录音状态下的按钮样式 */
.record-button.recording {
  background-color: #f44336; /* 红色背景 */
  transform: scale(1.05);    /* 轻微放大 */
}

/* 录音指示器容器 */
.recording-indicator {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: #f44336;           /* 红色文字 */
}

/* 脉冲动画效果 */
.pulse {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: #f44336;
  animation: pulse 1.5s infinite; /* 无限循环动画 */
}

/* 脉冲动画关键帧 */
@keyframes pulse {
  0% { transform: scale(0.95); opacity: 0.7; }
  70% { transform: scale(1.1); opacity: 1; }
  100% { transform: scale(0.95); opacity: 0.7; }
}

/* 回放区域样式 */
.playback {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  width: 100%;
  max-width: 300px;
}

/* 下载按钮样式 */
.download-button {
  padding: 0.5rem 1rem;
  background-color: #2196F3; /* 蓝色背景 */
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* 错误信息样式 */
.error-message {
  color: #f44336;           /* 红色文字 */
  text-align: center;
}
</style>
