# Onboarding full-stack-fastapi-template

## Cập nhật S4-06 ngày 2026-10-12

Baseline đã vá dependency: Trivy 0 finding; gate default PASSED_WITH_WARNINGS, exit 0, chỉ còn Low coverage.
Đã chuẩn hóa tuple exception cho parser Python 3.11; worker coverage-debt không còn lỗi cú pháp.
20 test login/items pass; harness S4-05 5 test noteboard + 6 test Node pass (kiểm image bằng nội dung Python).
C local BLOCKED; D local PASSED_WITH_WARNINGS; E local chỉ semgrep và PASSED. Đây chưa phải bằng chứng GitHub A–E.
LLM vẫn chưa bật; GT có guard allow_llm=false. B chờ teammate. D Jira giả tạo 2 ticket thay vì 1: cần sửa riêng trước Jira thật/10 lượt.

- Runbook: [docs/qc-s4-06.md](../docs/qc-s4-06.md).
- Baseline report: [report.md](local/runs/s4-06-baseline-final/report.md).
- Kết quả chuẩn bị: [preparation-results.json](local/s4-06/preparation-results.json).
- Intake/patch C,D/ledger nằm trong local/s4-06, Git bỏ qua.
- Image dùng cho workflow: ghcr.io/minhduc-it/qc-agent@sha256:95c55978ec52316e648c07c6f11ff780a930db3950d285504473d43965c6c785 (Private; GHCR_PULL_TOKEN).

Phần dưới là lịch sử trước bản vá baseline, không phải trạng thái hiện tại.

## Lịch sử onboarding ban đầu

Đã chạy thử local ngày 2026-10-12. QA owner: @MinhDuc-IT.
Image QC-Agent đã publish: `ghcr.io/minhduc-it/qc-agent@sha256:d4d447f94f362d21b0199dfa18196f15c1fa96fa6e11935e1b5772c7fdf1adea`. Manifest đã xác minh bằng credentials; package Private cần PAT; chưa chạy GitHub Actions.

## Kết quả thật

Smoke: health, OpenAPI, frontend HTTP, từ chối truy cập chưa đăng nhập, login admin, nhận diện admin và items CRUD đều pass.
Kiểm tra frontend ở đây chỉ là HTTP 200, chưa kiểm tra thao tác browser.

| Kịch bản | Verdict | Exit | Bằng chứng local |
|---|---|---|---|
| health-clean | PASSED | 0 | [report](local/runs/health-clean-20261012-102435/report.md) |
| health-mutant | BLOCKED | 1 | [report](local/runs/health-mutant-20261012-102438/report.md) |
| full | BLOCKED | 1 | [report](local/runs/full-20261012-102741/report.md) |

health-clean/health-mutant chỉ kiểm tra health endpoint. Mutant là container riêng có middleware trả HTTP 500; không sửa mã nguồn sản phẩm.
full dùng policy PR với api-contract, sast, secrets, deps, perf-smoke.
Lượt đầu có coverage-debt bị error: parser Python 3.11 của QC-Agent không đọc được cú pháp except của SUT Python 3.14.
Worker này được bỏ khỏi advisory_suites của policy project cho tới khi hỗ trợ Python 3.14; suite vẫn có sẵn để kiểm tra thủ công.

| Task trong full | Status |
|---|---|
| t-001 | pass |
| t-010 | pass |
| t-011 | pass |
| t-012 | fail |
| t-102 | pass |

Contract full chưa có token đăng nhập; smoke CRUD đã kiểm tra bằng tài khoản admin riêng. Chưa phải bộ test nghiệp vụ được QA duyệt.
Dependency scan lượt này báo target bun.lock; chưa có bằng chứng quét đủ dependency Python trong workspace uv.
Các lượt gate này không gọi LLM/GitHub/Jira. Thư mục local bị che khỏi cây mã nguồn trong container gate.

## Môi trường local

- App: http://localhost:18080 ; docs: http://localhost:18080/docs.
- Tài khoản admin: qc-admin@example.com. Mật khẩu nằm ở local/sut.env, không ghi trong tài liệu.
- local/db.env: thông tin PostgreSQL test. local/sut.env: env app và tài khoản test.
- local/ và nội dung của nó được Git/Docker build context bỏ qua. Report local cũng không commit.
- Container sạch: qc-fastapi-sut; DB riêng: qc-fastapi-db; mạng: qc-fastapi-local.
- Dockerfile.ci chạy migration và seed trước khi bật API; backend build và phục vụ frontend cùng image.

## Chạy lại (từ repo QC-Agent)

```powershell
docker run --rm --network qc-fastapi-local -e APP_BASE_URL=http://qc-fastapi-sut:8000 -e GIT_CONFIG_COUNT=1 -e GIT_CONFIG_KEY_0=safe.directory -e GIT_CONFIG_VALUE_0=/work -v "C:\VSF\full-stack-fastapi-template:/work:ro" --tmpfs /work/.qc-agent/local -v "C:\VSF\full-stack-fastapi-template\.qc-agent\local\runs:/reports" -v "${PWD}\configs\projects:/policy:ro" -w /work qc-agent:fastapi-local run --project full-stack-fastapi-template --mode pr --sut-root /work --projects-dir /policy --runs-dir /reports --suites api-contract,sast,secrets,deps,perf-smoke
```

Docker Desktop cần hoạt động; app/DB cần đang chạy. Muốn dừng môi trường: docker stop qc-fastapi-sut qc-fastapi-db.
Muốn mở lại: docker start qc-fastapi-db qc-fastapi-sut.

## Còn lại trước GitHub CI


- Xác nhận SHA QC-Agent đã có trên remote và phù hợp với image; workflow đang ghim SHA local lúc onboarding.
- Đưa policy configs/projects/full-stack-fastapi-template.yaml lên main QC-Agent qua review.
- Tạo secrets GitHub SUT_DB_SECRET_ENV và SUT_SECRET_ENV bằng thông tin CI riêng (không commit).
  DB host trong GitHub là db, khác qc-fastapi-db ở local.
- Chạy PR sạch/PR lỗi thật trên GitHub trước khi bật required check và branch protection.
- Ground-Truth hiện chỉ sinh thủ công; thống nhất PRD, ngân sách và provider trước khi bật LLM.

Không có thay đổi GitHub, Jira hoặc branch protection trong phiên local này.
