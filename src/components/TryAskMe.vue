<!--
  试试问我组件
  功能：
  1. 最大宽度 652px，默认常显
  2. 问题流式排列，flex-wrap 自动换行，最多显示 2 行
  3. 点击"换一换"或 10s 自动轮播，左出右入切换动效
  4. 鼠标悬停暂停轮播
  5. 每页数量由实际宽度动态计算，放不下就放到下一页
-->
<template>
  <div 
    class="try-ask-me" 
    @mouseenter="handleMouseEnter" 
    @mouseleave="handleMouseLeave"
  >
    <!-- 头部：标题 + 页码 + 换一换按钮 -->
    <div class="header">
      <span class="title">试试问我</span>
      <!-- 页码指示器：当前页/总页数  当前页条数 -->
      <span class="page-indicator">{{ currentPage + 1 }} / {{ totalPages }}  {{ currentQuestions.length }} 条</span>
      <button class="refresh-btn" @click="handleRefresh">
        <span class="refresh-icon">&#10227;</span>
        <span class="refresh-text">换一换</span>
      </button>
    </div>

    <!--
      隐藏的测量层：渲染全部问题，用于测量每个 tag 的实际宽度和换行位置
      visibility: hidden 不可见但占据布局空间，getBoundingClientRect 可获取准确位置
    -->
    <div ref="measureList" class="measure-list">
      <div 
        v-for="item in allQuestions" 
        :key="'m-' + item.text"
        class="question-tag"
      >
        <span class="question-text">{{ item.text }}</span>
        <span 
          class="category-tag"
          :class="'category-' + item.category"
        >
          {{ item.categoryName }}
        </span>
      </div>
    </div>

    <!-- 实际展示的问题区域 -->
    <div class="questions-container">
      <!--
        transition 实现左出右入动画：
        - leave: translateX(0)  translateX(-100%)，旧问题向左滑出
        - enter: translateX(100%)  translateX(0)，新问题从右滑入
        - currentKey 变化触发 transition 切换
      -->
      <transition name="slide-fade">
        <div :key="currentKey" class="questions-list">
          <div 
            v-for="item in currentQuestions" 
            :key="item.text"
            class="question-tag"
          >
            <span class="question-text">{{ item.text }}</span>
            <!-- 分类标签：办公/消保/服务/法律，不同颜色区分 -->
            <span 
              class="category-tag"
              :class="'category-' + item.category"
            >
              {{ item.categoryName }}
            </span>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TryAskMe',
  data() {
    return {
      // 全部问题数据，按顺序排列
      allQuestions: [
        { text: '如何申请制作电子印章，具体怎么操作', category: 'office', categoryName: '办公' },
        { text: '金融营销宣传的"八不得"是什么', category: 'finance', categoryName: '消保' },
        { text: '如何以发文字形式向监管部门发送报告', category: 'office', categoryName: '办公' },
        { text: '收到监管部门交办的国家秘密文件，如何妥善处理', category: 'office', categoryName: '办公' },
        { text: '个人征信报告怎么查询', category: 'finance', categoryName: '消保' },
        { text: '如何规范撰写会议纪要', category: 'office', categoryName: '办公' },
        { text: '客户投诉处理流程是什么', category: 'service', categoryName: '服务' },
        { text: '反洗钱工作要点有哪些', category: 'finance', categoryName: '消保' },
        { text: '公文格式有哪些规范要求', category: 'office', categoryName: '办公' },
        { text: '如何进行客户身份识别', category: 'finance', categoryName: '消保' },
        { text: '档案管理有哪些注意事项', category: 'office', categoryName: '办公' },
        { text: '理财产品风险等级如何划分', category: 'finance', categoryName: '消保' },
        { text: '会议通知的写作要点', category: 'office', categoryName: '办公' },
        { text: '如何处理客户紧急投诉', category: 'service', categoryName: '服务' },
        { text: '消费者权益保护法核心要点', category: 'law', categoryName: '法律' },
        { text: '印章使用管理规定是什么', category: 'office', categoryName: '办公' },
        { text: '如何防范电信网络诈骗', category: 'finance', categoryName: '消保' },
        { text: '服务礼仪规范有哪些', category: 'service', categoryName: '服务' },
        { text: '合同审查要点有哪些', category: 'law', categoryName: '法律' },
        { text: '如何撰写工作汇报材料', category: 'office', categoryName: '办公' },
        { text: '客户信息保护措施是什么', category: 'finance', categoryName: '消保' },
        { text: '应急预案怎么编写', category: 'office', categoryName: '办公' },
        { text: '服务标准有哪些要求', category: 'service', categoryName: '服务' },
        { text: '民间借贷相关法律规定', category: 'law', categoryName: '法律' },
        { text: '如何申请电子发票', category: 'office', categoryName: '办公' },
        { text: '可疑交易报告怎么报送', category: 'finance', categoryName: '消保' },
        { text: '客户满意度提升方法', category: 'service', categoryName: '服务' },
        { text: '劳动法常见问题解答', category: 'law', categoryName: '法律' },
        { text: '如何规范使用公章', category: 'office', categoryName: '办公' },
        { text: '理财产品销售规范是什么', category: 'finance', categoryName: '消保' },
        { text: '如何处理客户异议', category: 'service', categoryName: '服务' },
        { text: '合同签订注意事项有哪些', category: 'law', categoryName: '法律' },
        { text: '请示报告怎么写', category: 'office', categoryName: '办公' },
        { text: '客户身份资料如何保管', category: 'finance', categoryName: '消保' },
        { text: '服务投诉处理时限要求', category: 'service', categoryName: '服务' },
        { text: '知识产权保护相关规定', category: 'law', categoryName: '法律' }
      ],
      // 当前页展示的问题列表
      currentQuestions: [],
      // 用于触发 transition 动画的 key，每次翻页 +1
      currentKey: 0,
      // 当前页码（0-based）
      currentPage: 0,
      // 分页结果：二维数组，每个元素是一页的问题索引数组
      pages: [],
      // 动画锁，防止动画期间重复触发翻页
      isAnimating: false,
      // 自动轮播定时器
      timer: null,
      // 监听容器尺寸变化的 ResizeObserver 实例
      resizeObserver: null
    }
  },
  computed: {
    // 总页数
    totalPages() {
      return this.pages.length || 1
    }
  },
  mounted() {
    // DOM 渲染完成后，先测量分页再展示
    this.$nextTick(() => {
      this.buildPages()
      this.refreshQuestions()
      // 监听容器尺寸变化（窗口缩放等），重新计算分页
      this.resizeObserver = new ResizeObserver(() => {
        this.buildPages()
        this.refreshQuestions()
      })
      this.resizeObserver.observe(this.$refs.measureList)
    })
    // 启动 10s 自动轮播
    this.startTimer()
  },
  beforeDestroy() {
    // 组件销毁前清理定时器和观察器
    this.stopTimer()
    if (this.resizeObserver) {
      this.resizeObserver.disconnect()
    }
  },
  methods: {
    /**
     * 构建分页
     * 通过测量层中每个 tag 的 getBoundingClientRect().top 判断它在第几行
     * 每 2 行切一页，行号超过当前页起始行 + 2 时换页
     */
    buildPages() {
      const list = this.$refs.measureList
      if (!list) return
      const tags = list.querySelectorAll('.question-tag')
      if (tags.length === 0) return

      // 单行高度 = tag 高度 + gap
      const tagHeight = tags[0].offsetHeight
      const gap = 12
      const rowHeight = tagHeight + gap

      const pages = []
      let currentPageTags = []
      let pageStartRow = 0 // 当前页的起始行号

      for (let i = 0; i < tags.length; i++) {
        const top = tags[i].getBoundingClientRect().top
        // 计算当前 tag 在第几行（相对于第一个 tag 的偏移 / 行高）
        const row = Math.round((top - tags[0].getBoundingClientRect().top) / rowHeight)

        // 行号 >= 起始行 + 2，说明进入第三行了，需要换页
        if (row >= pageStartRow + 2) {
          pages.push(currentPageTags)
          currentPageTags = []
          pageStartRow = row
        }
        currentPageTags.push(i)
      }
      // 最后一页
      if (currentPageTags.length > 0) {
        pages.push(currentPageTags)
      }

      this.pages = pages
      // 如果当前页码超出新总页数，回到第一页
      if (this.currentPage >= pages.length) {
        this.currentPage = 0
      }
    },

    // 根据当前页码，从 pages 中取出对应索引，映射为实际数据
    refreshQuestions() {
      if (this.pages.length === 0) {
        this.currentQuestions = []
        return
      }
      const indices = this.pages[this.currentPage]
      this.currentQuestions = indices.map(i => this.allQuestions[i])
      // key 变化触发 transition 动画
      this.currentKey++
    },

    // 翻到下一页，循环
    nextPage() {
      this.currentPage = (this.currentPage + 1) % this.totalPages
    },

    // 换一换：先触发动画，动画结束后切换数据
    handleRefresh() {
      if (this.isAnimating) return
      this.isAnimating = true
      setTimeout(() => {
        this.nextPage()
        this.refreshQuestions()
        this.isAnimating = false
      }, 800)
    },

    // 鼠标悬停：暂停自动轮播
    handleMouseEnter() {
      this.stopTimer()
    },

    // 鼠标离开：恢复自动轮播
    handleMouseLeave() {
      this.startTimer()
    },

    // 启动 10s 自动轮播
    startTimer() {
      this.stopTimer()
      this.timer = setInterval(() => {
        this.handleRefresh()
      }, 10000)
    },

    // 停止自动轮播
    stopTimer() {
      if (this.timer) {
        clearInterval(this.timer)
        this.timer = null
      }
    }
  }
}
</script>

<style scoped>
/* ========== 容器 ========== */
.try-ask-me {
  width: 100%;
  max-width: 652px;           /* 最大宽度 652px */
  padding: 20px 24px;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
  box-sizing: border-box;
  position: relative;          /* 为测量层提供定位父级 */
}

/* ========== 头部 ========== */
.header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;
}

.title {
  font-size: 18px;
  font-weight: 700;
  color: #1a1a1a;
}

/* 页码指示器：等宽数字，不可选中 */
.page-indicator {
  font-size: 13px;
  color: #999;
  margin-left: auto;           /* 推到最右 */
  margin-right: 12px;
  font-feature-settings: "tnum"; /* 等宽数字 */
  user-select: none;
}

/* ========== 换一换按钮 ========== */
.refresh-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 0;
  background: none;
  border: none;
  font-size: 14px;
  color: #999;
  cursor: pointer;
  transition: color 0.2s ease;
}

.refresh-btn:hover {
  color: #666;
}

.refresh-btn:active {
  transform: scale(0.98);
}

.refresh-icon {
  font-size: 14px;
}

/* ========== 问题展示区域 ========== */
.questions-container {
  width: 100%;
  overflow: hidden;            /* 隐藏溢出（第三行及以后的内容） */
  height: 80px;                /* 固定高度，正好容纳两行 */
  position: relative;
}

.questions-list {
  display: flex;
  flex-wrap: wrap;             /* 自动换行 */
  gap: 12px;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}

/* ========== 隐藏的测量层 ========== */
/* 与 questions-list 完全相同的布局参数，用于测量实际换行位置 */
.measure-list {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  width: calc(100% - 48px);    /* 匹配实际内容区宽度（减去 padding） */
  visibility: hidden;          /* 不可见但占布局空间 */
  position: absolute;
  top: 0;
  left: 24px;                  /* 对齐内容区起始位置 */
  pointer-events: none;        /* 不响应鼠标事件 */
  height: auto;
}

/* ========== 切换动画：左出右入 ========== */
.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: transform 0.8s ease, opacity 0.8s ease;
}

/* 新问题从右侧滑入 */
.slide-fade-enter {
  transform: translateX(100%);
  opacity: 0;
}

/* 旧问题向左滑出 */
.slide-fade-leave-to {
  transform: translateX(-100%);
  opacity: 0;
}

/* ========== 问题标签 ========== */
.question-tag {
  display: inline-flex;
  align-items: center;
  padding: 7px 12px 7px 14px;
  background: #ffffff;
  border: 1px solid #e8e8e8;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.2s ease;
  max-width: 100%;
  height: 32px;
  box-sizing: border-box;
  flex-shrink: 0;              /* 不压缩，按实际宽度排列 */
}

.question-tag:hover {
  border-color: #d0d0d0;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}

/* 问题文本：不换行，不截断，按实际长度 */
.question-text {
  font-size: 13px;
  color: #333;
  white-space: nowrap;
}

/* ========== 分类标签 ========== */
.category-tag {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  margin-left: 8px;
  padding: 2px 6px;
  font-size: 12px;
  border-radius: 4px;
  flex-shrink: 0;              /* 不压缩，始终显示 */
  font-weight: 500;
}

/* 办公 - 蓝色 */
.category-office {
  background: #e6f4ff;
  color: #2b7adb;
}

/* 消保 - 紫色 */
.category-finance {
  background: #f3e8ff;
  color: #7c3aed;
}

/* 服务 - 绿色 */
.category-service {
  background: #ecfdf5;
  color: #059669;
}

/* 法律 - 黄色 */
.category-law {
  background: #fef3c7;
  color: #d97706;
}
</style>
