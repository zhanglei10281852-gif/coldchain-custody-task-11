# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

同一传感器序号重复上报时数据库拒绝写入，但 API 最终返回内部错误，调用方无法识别为可预期的业务冲突。请修复错误链传播，保留冲突分类。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-11
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-11.git
- parent SHA：b60785a962283546d6587af6b175b2ebcf708cfc

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-11.git bug-repro
cd bug-repro
git checkout --detach b60785a962283546d6587af6b175b2ebcf708cfc
go test ./internal/service -run "^TestDuplicateReadingPreservesConflict$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestDuplicateReadingPreservesConflict$" -count=1
--- FAIL: TestDuplicateReadingPreservesConflict (0.81s)
    annotation_behavior_test.go:213: duplicate reading error = insert temperature reading failed: constraint failed: UNIQUE constraint failed: temperature_readings.shipment_id, temperature_readings.sensor_id, temperature_readings.sequence (2067)
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	0.822s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestDuplicateReadingPreservesConflict$" -count=1
--- FAIL: TestDuplicateReadingPreservesConflict (2.46s)
    annotation_behavior_test.go:213: duplicate reading error = insert temperature reading failed: constraint failed: UNIQUE constraint failed: temperature_readings.shipment_id, temperature_readings.sensor_id, temperature_readings.sequence (2067)
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	2.680s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令 go test ./internal/service -run ^TestDuplicateReadingPreservesConflict$ -count=1 必须由修复前失败变为修复后通过；相关包与 go test ./... -count=1 全量回归通过，回退 gold 关键修改后定向命令重新失败。
