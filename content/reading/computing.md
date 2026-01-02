---
comment: false
layout: "single"
quote: "Reading - 📖读书破万卷"
reading_nav:
  - title: "我的"
    url: "/reading/mine/"
  - title: "心理学"
    url: "/reading/psychology/"
  - title: "哲学"
    url: "/reading/philosophy/"
  - title: "计算机科学"
    url: "/reading/computing/"
  - title: "工具"
    url: "/reading/manuals/"
---

<style>
  .book-list {
    list-style: none;
    padding: 0;
  }
  .book-item {
    display: flex;
    align-items: center;
    margin-bottom: 20px;
    border: 1px solid #ddd;
    padding: 10px;
    border-radius: 5px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    transition: background-color 0.3s ease, box-shadow 0.3s ease, transform 0.3s ease;
  }
  .book-item:hover {
    background-color: #f9f9f9;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    transform: scale(1.05);
  }
  .book-cover {
    flex-shrink: 0;
    width: 100px;
    height: 150px;
    object-fit: contain; /* 确保图片内容全部显示 */
    border-radius: 5px;
    margin-right: 15px;
  }
  .book-details {
    flex: 1;/*保证详情区域宽度自适应 */
    padding: 10px;
  }
  .book-title {
    font-size: 1.1em;
    margin: 0 0 10px;
  }
  .book-meta {
    font-size: 0.9em;
    color: #777;
    margin-bottom: 5px;
  }
  .book-description {
    font-size: 0.85em;
    color: #333;
    line-height: 1.4;
  }
  .book-rating {
    color: #f39c12;
  }
  .book-tags {
    font-size: 0.9em;
    color: #555;
    margin-bottom: 5px;
  }
  .book-tags span {
    background-color: #f0f0f0;
    border-radius: 3px;
    padding: 3px 6px;
    margin-right: 5px;
  }
</style>

<ul class="book-list">
  <li class="book-item">
    <img src="/images/reading/shenruqianchumysql.jpg" alt="深入浅出MySQL" class="book-cover" />
    <div class="book-details">
      <div class="book-title">深入浅出MySQL</div>
      <div class="book-meta">类目：计算机</div>
      <div class="book-rating">评分：⭐⭐⭐⭐⭐</div>
      <div class="book-tags"><span>MySQL</span></div>
      <div class="book-description"></div>
    </div>
  </li>
  <li class="book-item">
    <img src="/images/reading/goyuyanquxuezhinan.jpg" alt="Go语言趣学指南" class="book-cover" />
    <div class="book-details">
      <div class="book-title">Go语言趣学指南</div>
      <div class="book-meta">类目：计算机</div>
      <div class="book-rating">评分：⭐⭐⭐⭐⭐</div>
      <div class="book-tags"><span>Go</span></div>
      <div class="book-description"></div>
    </div>
  </li>
</ul>
