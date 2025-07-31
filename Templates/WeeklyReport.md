<%*
// 安全な週報タイトルを生成
const year = tp.date.now("YYYY");
const weekNumber = tp.date.now("WW");
const title = `${year}年 第${weekNumber}週 週報`.replace(/[\\/:*?"<>|]/g, "");
await tp.file.rename(title);
-%>
---
title: <%* tR += title %>
tags:
  - weekly-report
week: <%* tR += `${year}-W${weekNumber}` %>
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
---

# <%* tR += title %>

**期間:** <% tp.date.weekday("YYYY-MM-DD", 0, title, "YYYY年 第WW週 週報") %> ~ <% tp.date.weekday("YYYY-MM-DD", 6, title, "YYYY年 第WW週 週報") %>

---

##  Keep (良かったこと・継続したいこと)

- <% tp.file.cursor() %>

##  Problem (問題点・改善したいこと)

- 

##  Try (次に試すこと)

- 

---

##  今週の学び・気づき

- 

##  その他・雑感

-
