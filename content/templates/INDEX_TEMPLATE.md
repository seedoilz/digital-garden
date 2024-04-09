<%*
const dv = app.plugins.plugins["dataview"].api;
const filename = "index";
const query1 = `LIST FROM #project SORT file.mtime DESC`;
const query2 = `TABLE WITHOUT ID replace(string(file.link), "|" + file.name, "") AS "File", file.frontmatter["date modified"] AS "Last Edited Time", tags AS "tags" 
WHERE file.name != "index" AND file.name != "INDEX_TEMPLATE"
SORT file.mtime DESC LIMIT 10`;
console. log

let createdTime = "2024-03-19 21:03:00"
let modifiedDate = moment().format("YYYY-MM-DD HH:MM:SS");

const linter = `---
aliases: 
title: index
date created: ${createdTime}
date modified: ${modifiedDate}
tags: 
---\n`;
const tFile = tp.file.find_tfile(filename);
const queryOutput1 = await dv.queryMarkdown(query1);
const queryOutput2 = await dv.queryMarkdown(query2);

const queryOutput = linter + '## Projects\n' + queryOutput1.value + '## Recently Edited\n' + queryOutput2.value

await app.vault.modify(tFile, queryOutput);
%>