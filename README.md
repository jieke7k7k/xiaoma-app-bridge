# 小马课表 · AI 桥信箱

这是「小马课表」App 与外部 AI 之间的命令信箱（私有仓库）。外部 AI 往这里放指令文件，
App 在启动/手动同步时读取、执行，并把结果与课表快照写回。

## 文件布局

- `commands/<id>.json` — 指令批（外部 AI 写）。**文件名（去掉 .json）就是这批的规范 id**，
  正文里的 `id` 仅作参考，两者不一致时 App 以文件名为准。id 只能用字母、数字、下划线、连字符。
- `results/<id>.json` — 执行结果（App 写回），id 与指令批文件名一致。
- `state/snapshot.json` — 课表快照（App 每次同步后更新）：当前学期、全部课程精简字段、
  按日期的备注、考试日期。回答用户课表问题时先看它。

## 指令批格式

```json
{
  "id": "20261022-001",
  "createdAt": "2026-10-22T15:00:00+08:00",
  "note": "可选的人类可读说明",
  "commands": [
    { "type": "add_note", "date": "2026-10-23", "content": "交作业" }
  ]
}
```

## 一期指令集

| type | 参数 | 破坏性 |
|---|---|---|
| add_course | name, dayOfWeek(1-7), startSection, endSection, weeks(如 "1-16" 或 "1,3,5"), weekType(all/odd/even), teacher?, classroom?, note?, colorValue? | 否 |
| set_course_color | courseId 或 courseName（重名批量全改）, color 或 colorValue | 否 |
| add_note | date(yyyy-MM-dd), content | 否 |
| delete_note | id（备注 id，先从 snapshot 里查） | **是** |
| add_exam | courseId 或 courseName, date(yyyy-MM-dd), time?(HH:mm) | 否 |

安全规则：非破坏指令 App 自动执行；破坏指令（delete_note）不会自动执行，
会进入 App 内「待确认」队列，用户点「执行」后才落库，结果里 state 为
executed / rejected / pending 三态。整批状态 status：done / partial / awaiting_confirm。

## results 格式

```json
{
  "id": "20261022-001",
  "executedAt": "2026-10-22T15:05:00+08:00",
  "status": "done",
  "results": [
    { "index": 0, "type": "add_note", "success": true, "message": "已添加", "state": "executed" }
  ]
}
```

## 令牌

App 侧使用仅限本仓库的细粒度令牌（Contents 读写），只保存在用户手机本地，
不写进本仓库任何文件。不要把令牌提交到这里。
