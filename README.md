# ai-eng-labs

Repo thực hành theo lộ trình **AI Engineering — từ sinh viên đến năng lực senior** (15 tháng, 8 module).
Mỗi lab là một thư mục riêng, có README ghi **mục tiêu · cách chạy · kết quả đo**.

📄 Lộ trình gốc: [docs/Lo-trinh-AI-Engineering.pdf](docs/Lo-trinh-AI-Engineering.pdf)

## Cách học một khối 2 tuần

- **Lý thuyết (~30%)**: đọc tài liệu, tự viết 1 trang tóm tắt vào `notes/`.
- **Lab (~60%)**: code trong thư mục lab, commit thường xuyên, ghi số đo vào README của lab.
- **Tự kiểm tra (~10%)**: trả lời câu hỏi cuối module trong `self-check.md`.

Chỉ sang module tiếp khi đã tick hết chuẩn đầu ra, không phải khi hết thời gian.

## Tiến độ

⬜ chưa bắt đầu · 🟨 đang làm · ✅ đạt

| Module | Lab | Khối | Trạng thái |
|---|---|---|---|
| M1 | [01-doc-loader](m1-software-foundations/01-doc-loader/) | Khối 1 · Tuần 1–2 · Python production | ⬜ |
| M1 | [02-regulation-api](m1-software-foundations/02-regulation-api/) | Khối 2 · Tuần 3–4 · FastAPI và PostgreSQL | ⬜ |
| M1 | [03-redis-queue-reliability](m1-software-foundations/03-redis-queue-reliability/) | Khối 3 · Tuần 5–6 · Redis, hàng đợi, độ tin cậy | ⬜ |
| M1 | [04-docker-ci-deploy](m1-software-foundations/04-docker-ci-deploy/) | Khối 4 · Tuần 7–8 · Docker, CI/CD, deploy | ⬜ |
| M2 | [01-numpy-vector-search](m2-ml-dl-foundations/01-numpy-vector-search/) | Khối 1 · Tuần 6–7 · Toán cốt lõi | ⬜ |
| M2 | [02-vn-question-classifier](m2-ml-dl-foundations/02-vn-question-classifier/) | Khối 2 · Tuần 8–9 · ML cổ điển và thống kê cho eval | ⬜ |
| M2 | [03-micro-autograd-pytorch](m2-ml-dl-foundations/03-micro-autograd-pytorch/) | Khối 3 · Tuần 10–11 · Deep learning với PyTorch | ⬜ |
| M2 | [04-char-gpt-vietnamese](m2-ml-dl-foundations/04-char-gpt-vietnamese/) | Khối 4 · Tuần 12–14 · Transformer từ bên trong | ⬜ |
| M2 | [05-llm-sampling](m2-ml-dl-foundations/05-llm-sampling/) | Khối 5 · Tuần 15–16 · LLM trong thực tế | ⬜ |
| M3 | [01-llm-client](m3-llm-app-rag/01-llm-client/) | Khối 1 · Tuần 16–17 · LLM API và context engineering | ⬜ |
| M3 | [02-rag-v1](m3-llm-app-rag/02-rag-v1/) | Khối 2 · Tuần 18–19 · RAG v1 end-to-end | ⬜ |
| M3 | [03-hybrid-retrieval](m3-llm-app-rag/03-hybrid-retrieval/) | Khối 3 · Tuần 20–21 · Retrieval nâng cao | ⬜ |
| M3 | [04-rag-v2-citations](m3-llm-app-rag/04-rag-v2-citations/) | Khối 4 · Tuần 22–24 · Generation, UX và v2 | ⬜ |
| M4 | [01-golden-set-error-analysis](m4-evals-observability/01-golden-set-error-analysis/) | Khối 1 · Tuần 24–25 · Golden set và phân tích lỗi | ⬜ |
| M4 | [02-eval-harness](m4-evals-observability/02-eval-harness/) | Khối 2 · Tuần 26–27 · Metric và LLM-as-judge | ⬜ |
| M4 | [03-eval-in-ci](m4-evals-observability/03-eval-in-ci/) | Khối 3 · Tuần 28–29 · Eval trong CI và vòng lặp thử nghiệm | ⬜ |
| M4 | [04-tracing-online-eval](m4-evals-observability/04-tracing-online-eval/) | Khối 4 · Tuần 30–32 · Observability và eval online | ⬜ |
| M5 | [01-agent-loop-from-scratch](m5-agents-mcp/01-agent-loop-from-scratch/) | Khối 1 · Tuần 32–33 · Tool calling và vòng lặp agent tự viết | ⬜ |
| M5 | [02-workflow-vs-agent](m5-agents-mcp/02-workflow-vs-agent/) | Khối 2 · Tuần 34–35 · Workflow hay agent, và thiết kế tool | ⬜ |
| M5 | [03-mcp-server](m5-agents-mcp/03-mcp-server/) | Khối 3 · Tuần 36–37 · MCP, trạng thái và con người trong vòng lặp | ⬜ |
| M5 | [04-agent-eval](m5-agents-mcp/04-agent-eval/) | Khối 4 · Tuần 38–40 · Eval agent và framework | ⬜ |
| M6 | [01-reliability-fault-injection](m6-production-llmops-security/01-reliability-fault-injection/) | Khối 1 · Tuần 40–41 · Độ tin cậy | ⬜ |
| M6 | [02-cost-dashboard](m6-production-llmops-security/02-cost-dashboard/) | Khối 2 · Tuần 42–43 · Chi phí | ⬜ |
| M6 | [03-load-test](m6-production-llmops-security/03-load-test/) | Khối 3 · Tuần 44–45 · Tải và mở rộng | ⬜ |
| M6 | [04-red-team-guardrails](m6-production-llmops-security/04-red-team-guardrails/) | Khối 4 · Tuần 46–48 · Bảo mật và guardrails | ⬜ |
| M6 | [05-canary-incidents](m6-production-llmops-security/05-canary-incidents/) | Khối 5 · Tuần 49–52 · Vận hành và sự cố | ⬜ |
| M7 | [01-finetune-decision](m7-finetune-selfhost/01-finetune-decision/) | Khối 1 · Tuần 50–51 · Khi nào fine-tune, và dữ liệu | ⬜ |
| M7 | [02-lora-router](m7-finetune-selfhost/02-lora-router/) | Khối 2 · Tuần 52–54 · LoRA cho model định tuyến | ⬜ |
| M7 | [03-embedding-finetune](m7-finetune-selfhost/03-embedding-finetune/) | Khối 3 · Tuần 55–57 · Fine-tune embedding cho miền quy chế | ⬜ |
| M7 | [04-vllm-serving](m7-finetune-selfhost/04-vllm-serving/) | Khối 4 · Tuần 58–60 · Self-hosting | ⬜ |
| M8 | [senior skills](m8-senior-skills/) | Xuyên suốt | ⬜ |

## Cấu trúc

```
ai-eng-labs/
├── m1-…/ … m7-…/      # mỗi module: README (chuẩn đầu ra), self-check.md, các lab
│   └── 01-<lab>/      # README.md + src/
├── m8-senior-skills/  # system design, ADR, blog, postmortem log
├── notes/             # tóm tắt lý thuyết bằng lời của mình
└── docs/              # lộ trình gốc
```

> Dự án chính (RAG quy chế HUCE, lớn dần qua M3 → M6) nên để ở repo riêng; ADR của dự án nằm trong `docs/adr/` của repo đó.
