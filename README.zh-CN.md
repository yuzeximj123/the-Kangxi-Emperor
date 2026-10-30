<p align="center">
  <img src="assets/cover.jpg" alt="清圣祖康熙皇帝朝服像" height="520">
</p>

<h1 align="center">康熙朝治理史料库</h1>

<p align="center">
  <b>简体中文</b> · <a href="README.md">English</a>
</p>

把**康熙皇帝（1661–1722 在位）**如何治理国家的第一手史料——每日的起居注，以及他亲笔批写的朱批奏折——数字化、结构化，整理成一个可以直接查询、可以跨文献互相链接的事件数据库。

> 封面：《清圣祖康熙皇帝朝服像》，清代宫廷绘画。

## 看点：数据里有什么

先看几个大家最关心的结论（数字均由 `events.db` 统计，使用前请读[已知局限](#已知局限)）。

**数字里的皇帝日常。** 54,401 条事件中，最多的是皇帝行止（7,772）、议政听政（6,773）、谕旨决策（6,095）和人事任免（5,193）。司法案件（2,946）是军务（1,302）的两倍多：这个朝廷多数时候在处理案牍，而不是打仗。

**谁出场最多。** 除康熙帝本人（出现 30,043 次）和太皇太后（1,209 次）外，出场最多的是几位大学士：明珠（2,016）、伊桑阿（1,884）、库勒纳（1,336）、张玉书（1,209）、马齐（1,077）。

**国家在什么时候最忙。**

| 主题 | 高峰年份 | 事件数 |
|---|---|---|
| 军务 | 康熙三十五年（1696）、三十六年（1697），即亲征噶尔丹 | 241、118 |
| 河工水利 | 三十九年（1700），其次二十七年（1688） | 82、58 |
| 人事任免 | 十九至二十五年，即 1680 年代的巩固期 | 每年 239–281 |

仅"噶尔丹"一词就出现在 440 条事件摘要中。

**皇帝自己的声音。** 6,193 条事件带有朱批，朱批摘要保留了他对臣下的口气。最常见的套话是报平安："朕体安"。几个例子：

- 三十二年，看到陕西收成报告："郎中常舒来奏陕西粮食甚好，朕甚喜悦。"
- 四十五年，行围："直到十三日仍未下霜，鹿亦未鸣。今朕已始行围猎，只是蚊子多受不得。"
- 四十五年，论知府任用："知府乃朕签补授之员，此二人朕不知，若听尔言用之可乎？"
- 五十八年，对送礼的官员："勿再向朕献物，尔若有需用之物务寄信。"

雨水与收成是反复出现的话题：有 1,168 件带朱批的奏折提到雨。

**长链案件。** `event_threads` 把同一件事在三种文献里串起来。最长的第 3 号链有 361 条事件，是康熙决意亲征噶尔丹的过程。其他还有噶礼与张伯行互参案（第 2、17、22 号链）、赵申乔父子贪劣之争（第 7 号链）、福建沿海盗乱奏报（第 4 号链）。可以这样看：

```sql
SELECT thread_id, member_count, corpora, label
FROM event_threads ORDER BY member_count DESC LIMIT 10;
```

**事情发生在哪里。** 除宫门乾清门（4,285 条）外，出现最多的地点是江南（945）、畅春园（892）、直隶（886）、福建（842）、浙江（838）：地方事务集中在长江下游和沿海。

## 仓库里有什么

三类原始史料，经过 OCR、清洗和结构化抽取：

| 史料 | 内容 | 册数 |
|---|---|---|
| 起居注 | 皇帝每日临朝、谕旨、任免、行止的记录 | 8 册 |
| 汉文朱批奏折汇编 | 官员奏折及皇帝的朱批，涵盖康熙 28–61 年 | 8 册 |
| 满文朱批奏折全译 | 原为满文的奏折及朱批的汉译 | 1 部（体量很大） |

最终产物是 SQLite 数据库 `event/events.db`，建立在清洗后的文本之上。

### 数据库内容（`event/events.db`）

| 表 | 行数 | 说明 |
|---|---:|---|
| `events` | 54,401 | 每行一个事件：起居注 46,316 条，满文奏折 5,025 条，汉文奏折 3,060 条。含日期、事件角色、摘要、朱批摘要 |
| `people` / `person_aliases` | 26,526 / 27,979 | 规范人名及别名；同名人物拆分见 `person_splits`、`person_identity_notes` |
| `event_people` | 146,409 | 每个事件涉及哪些人，以及各自的角色 |
| `person_posts` | 111,432 | 每个人随时间担任的官职，含规范化官名和类别 |
| `places` / `event_places` | 10,771 / 72,178 | 规范地名及事件发生地 |
| `event_links` | 625,455 | 事件之间的候选关联，含关系类型和置信度 |
| `event_threads` / `event_thread_members` | 213 / 1,290 | 跨文献的事件链，例如一件案子从最初奏报到最终结果 |

事件角色包括：皇帝行止、议政听政、谕旨决策、人事任免、司法案件、河工水利、军务等。

## 目录结构

```
assets/                  封面图
raw/                     原始 PDF（不入库，见"史料来源"）
processed/               各史料的 OCR 结果和目录
output/
  原稿/                   OCR 原始文本
  简体中文版/              清洗、标点后的简体中文文本
  现代白话文/              现代白话文译写
event/
  events.db              事件数据库（用 Git LFS 存储）
  export/                events.csv 与 events.json 导出
  staging/               各史料的抽取中间结果（JSON）
_archive/                临时文件与旧日志
```

## 快速开始

数据库用 [Git LFS](https://git-lfs.com/) 存储，克隆前请先安装。

```bash
git lfs install
git clone https://github.com/yuzeximj123/the-Kangxi-Emperor.git
cd the-Kangxi-Emperor
```

用任意 SQLite 工具即可查询：

```bash
sqlite3 event/events.db
```

```sql
-- 按康熙年份统计人事任免数量
SELECT reign_year, COUNT(*) FROM events
WHERE event_role = '人事任免' GROUP BY reign_year ORDER BY reign_year;

-- 某一时期出现次数最多的人物
SELECT p.canonical_name, COUNT(*) AS n
FROM event_people ep
JOIN events e USING (event_id)
JOIN people p USING (person_id)
WHERE e.reign_year BETWEEN 21 AND 30
GROUP BY 1 ORDER BY n DESC LIMIT 10;

-- 提到下雨的朱批
SELECT reign_year, zhupi_summary FROM events
WHERE has_zhupi = 1 AND zhupi_summary LIKE '%雨%';

-- 按时间顺序查看一条事件链
SELECT e.source_corpus, e.reign_year, e.month, e.summary
FROM event_thread_members m JOIN events e USING (event_id)
WHERE m.thread_id = 7
ORDER BY e.reign_year, e.month, e.order_index;
```

在 Windows 终端里用 Python 读取时，请设置 `PYTHONIOENCODING=utf-8`，否则中文会显示成乱码。

## 已知局限

使用数据得出结论之前，请先阅读这一节。

- **OCR 与抽取错误。** 文本来自扫描件的 OCR 和自动结构化抽取，人名、日期、官职都可能有误。
- **同名人物。** 同名的人可能被合并成一条记录。已有一部分被拆分（见 `person_splits`），但并不完整。数据里能看到的例子：某人几十年后才担任的职务，被记到了他早年的名下。
- **年份标注。** 部分记录的 `reign_year` 缺失或有误。例如满文部分在康熙 3 年有一个异常集中的数据块。
- **事件关联由模型推断。** `event_links` 和 `event_threads` 是推断出来的，平均置信度约 0.8，请当作线索，用原文核对后再下结论。
- **统计只是大致数字。** 对摘要做关键词计数时，一条记录可能同时命中多个词，数字只能说明倾向。
- **各时期覆盖不同。** 起居注实际从康熙 9–10 年起才有大量记录，汉文奏折从康熙 28 年起。

## 史料来源

原始文本是中国第一历史档案馆编的奏折汇编，以及康熙朝起居注的整理出版本。原始 PDF **不**在本仓库中（`raw/` 已被 git 忽略）。这些汇编的权利属于其编者和出版者；本仓库中的转录文本仅供研究使用。

## 许可

本仓库中的代码和衍生数据以 [GNU Affero 通用公共许可证 v3.0](LICENSE) 发布。该许可证不改变原始史料编者和出版者所享有的权利。
