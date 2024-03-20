<%*
const dv = app.plugins.plugins["dataview"].api;
const filename = "index";
const query1 = `LIST FROM #project`;
const query2 = `TABLE file.mtime AS "Last Modified", tags AS "tags" 
WHERE file.name != "index" AND file.name != "INDEX_TEMPLATE"
SORT file.mtime DESC LIMIT 10`;

let createdTime = "三月 20日 2024, 10:26:03 上午"
let modifiedDate = moment().format("MMMM Do YYYY, h:mm:ss a");

const linter = `---
aliases: 
title: index
date created: ${createdTime}
date modified: ${modifiedDate}
tags: [project, catalogue]
---\n`;
const tFile = tp.file.find_tfile(filename);
const queryOutput1 = await dv.queryMarkdown(query1);
const queryOutput2 = await dv.queryMarkdown(query2);

const queryOutput = linter + '## Projects\n' + queryOutput1.value + '## Recently Edited\n' + queryOutput2.value

await app.vault.modify(tFile, queryOutput);
%>