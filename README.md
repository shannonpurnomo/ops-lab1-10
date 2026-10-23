# enroll-conflict：实验 1 的统一示例

> 实验要求、任务步骤、检查点和评分见 Canvas 上发布的《实验 1：从提交到可追溯制品的流水线》指导书。本仓库只提供示例代码，不含指导书。

一个仅依赖标准库的小型服务：判断两个教学班的上课时间是否冲突，对应 L-05 课程实践中“冲突检测新规则”的场景。**服务本身不是实验的重点**，它的作用是为流水线提供一个可构建、可测试、可使测试失败的对象。

## 规则

两个教学班冲突，当且仅当：同一个星期、上课周次有交集、节次区间有重叠。上课周次分三种：`all`（每周）、`odd`（单周）、`even`（双周）。单周班与双周班在同一时段上课不冲突。

## 接口

| 方法与路径 | 作用 |
|---|---|
| `GET /healthz` | 存活检查，返回 `ok` |
| `GET /version` | 返回版本号、提交号、构建号、Go 版本（JSON） |
| `POST /api/conflict` | 请求体 `{"a": 教学班, "b": 教学班}`，返回 `{"conflict": true/false}` |

教学班的字段：`id`、`weekday`（1～7）、`start`、`end`（节次，含）、`weeks`。

## 在本地运行

需要 Go 1.26 或更高版本；`make` 目标还需要 GNU make 和 GNU tar（Linux 或 WSL2）。

```bash
make fmt-check vet test                    # quality gate
go run . -addr 127.0.0.1:8080              # 直接运行
make package VERSION=1.0.0 BUILD_NO=1      # 打包到 dist/
dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict -version
```

`make package` 的参数 `VERSION`、`BUILD_NO`、`COMMIT` 是制品的三个追溯标识，`RUN_ID` 用于指向具体的那次运行；未传入时分别取 `dev`、`local`、本地 `HEAD` 和 `local`，表示“未经过流水线”。

## 查询运行版本与回退

在 Linux 或 WSL2 中运行已下载并校验通过的 Release 制品后，可以查询当前服务的版本。例如，服务通过 `-addr 127.0.0.1:18080` 启动时，在另一终端执行：

```bash
curl -fsS http://127.0.0.1:18080/version
```

将返回的 `version`、`commit`、`build_no` 和 `go` 与该制品目录中的 `BUILD_INFO` 对照，确认正在运行的版本。运行 Release 中的 `linux/amd64` 二进制文件需要 x86-64 Linux 环境。

如需追溯制品，在已更新的小组仓库根目录执行以下命令，将文件名替换为目标版本，并将对应的 `.sha256` 文件放在压缩包旁边：

```bash
git fetch origin --tags
bash scripts/trace.sh dist/release/enroll-conflict_1.0.0_linux_amd64.tar.gz . origin/main
```

脚本会校验附件、读取 `BUILD_INFO`、检查提交是否存在于主干历史，并给出提交页面和构建运行链接。若缺少 `.sha256` 文件，脚本会跳过校验和检查，因此实验时应同时保留两个附件。

回退时，先停止当前服务，再从已下载并校验通过的旧版本 Release 附件启动服务，无需重新编译源码。切换后再次查询 `/version`，核对版本号、提交号和构建号。实验记录应保留切换前后的输出，以及旧版本发布时记录的 SHA-256 和重新下载后计算的值。已发布的版本号不应删除或复用；修复代码应通过新的 PR 和版本发布。

## 文件

| 文件 | 说明 |
|---|---|
| `conflict.go`、`conflict_test.go` | 冲突规则及其测试 |
| `main.go`、`main_test.go` | HTTP 服务、版本信息及其测试 |
| `Makefile` | 本地与 CI 共用的构建入口 |
| `scripts/trace.sh` | 追溯脚本：由制品回答“它来自哪次提交” |
| `Dockerfile` | 选做 O1 使用 |
| `LAB1.md` | 小组的证据与分析记录，需要填写并提交 |

仓库中没有 `.github/workflows/`：流水线由小组在实验中自行编写。
