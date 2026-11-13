# codex-security 速查表 | codex-security Cheatsheet | Guía rápida de codex-security

**它不只报漏洞，还会验证这条到底能不能触发、再给你补丁——但先看第一屏的权限与成本口径，别裸奔开扫**

> 一页看懂 OpenAI 的 codex-security：官方出的漏洞扫描 CLI 与 TypeScript SDK。能扫整个仓库、指定路径或一次 diff，能验证候选发现、生成补丁、核验已有修复，还带 GitHub Action、容器批量扫与 SARIF/JSON/CSV 导出。官方文档站已有西语与葡语译版，所以这张不卖翻译，卖的是把权限边界、成本上限和扫前纪律收在一页的实战口径。

- 上游项目：[openai/codex-security](https://github.com/openai/codex-security)（2026-11-13 实测 12,941 ★ / 1,011 forks，Apache-2.0；55 个 release，2,047 个 commit，根目录只有英文 README）
- 平台：终端 CLI（`cs` / `codex-security`）、Codex 内的 Security 插件、TypeScript SDK、GitHub Action
- 本页结构：中文主文 → English → Español → Português
- 同系列：给 agent 用的审计方法论 skill 看 [security-audit 速查表](https://github.com/Ezra-Zhao/security-audit-cheatsheet-es-pt)；这张是成套的扫描 CLI 加 SDK 加 CI，两张互引不抢读者

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 第一屏·权限口径卡（先定死，再开扫）

| 口径 | 实际情况（原仓自述） |
|---|---|
| 本地扫描不需要 Cloud | Cloud 是可选连接：读已有 Cloud 发现才用得上；本地扫描没有 Cloud 也完整可用 |
| 部分发现要 Trusted Access | 一些网络安全类请求与受保护发现，需要 Trusted Access for Cyber 资格；没有资格，这部分功能用不了，不是报错、是权限门槛 |
| Daybreak 账号另走 opt-in | 有 Daybreak 资格的账号，要在 Codex 里打开对应开关、或在配置里显式选定 cyber 访问计划，才走那条线 |
| CLI 用你本人的系统权限跑 | 它以你当前账号的权限运行，不会停下来逐项问批准。扫之前先把无关凭据移出环境变量——CI 里靠 `OPENAI_API_KEY` 或 `CODEX_API_KEY` 注入，只给扫描进程，别把整套密钥留在环境里 |
| 模型后端不止 OpenAI | 扫描也支持 Amazon Bedrock（用 AWS 凭据、不需要另登 OpenAI）、OpenRouter、Fireworks AI、Cheaper Inference，各自配各自的密钥与受支持模型 |

## 第一屏·成本口径卡

| 口径 | 实际情况（原仓自述） |
|---|---|
| GitHub Action 示例上限 5 美元 | 官方示例 workflow 给每次扫描设了约 5 美元的估算上限，但原仓自述：进行中的请求可能超限——上限是刹车，不是封顶保证 |
| Cloud 按 token 用量计费 | 走 Cloud 的按 token 用量算钱，先看计费提示再跑大仓 |
| CLI 有成本与时长参数 | `cs scan --help` 里有成本上限与调查时长选项（如 `--max-time 15` 给标准扫描设 15 分钟的建议时长）；深度扫描另有自己的时长口径，先看帮助再跑 |

## 三条起步路，先选哪条

| 路 | 装法与第一条命令 | 适合谁 |
|---|---|---|
| 终端 CLI | `npm install --global @openai/codex-security` → `cs login` → 在仓库目录跑 `cs scan .`；扫完打开终端给出的 REPORT 路径看报告，`cs scans show` 找回最近一次扫描 | 想马上在真仓库上扫一次的人 |
| Codex 插件 | 在 Codex 会话里启用 Codex Security 插件，直接让它按工作流扫一个仓库（可指定聚焦认证与授权这类面），报告与发现会存下来，之后再让它出补丁或核验已有修复 | 日常已在 Codex 里干活的人 |
| 自动化 | TypeScript SDK（项目内 `npm install @openai/codex-security`，用 `CodexSecurity` 类编排）或 GitHub Action（把密钥存为仓库 secret `CODEX_SECURITY_API_KEY`，示例 workflow 里的分发 commit SHA 换成已审过的版本再启用） | 要进 CI、批量扫多仓的人 |

短名提示：`cs` 与 `codex-security` 是同一个 CLI；短名和别的命令冲突就用全名。不想全局装、或 npm 报 `EEXIST` 冲突，用 `npx @openai/codex-security` 代替 `cs` 跑登录与扫描。

## 环境门槛一行版

Node.js 22.13.0 以上（限 22.x 线）或 24.x 或 26.x；Python 3.10 以上，其中 3.10 还要额外装 `tomli`。版本不在这几条线上的，先升级再装，免得装完跑不起来还以为是工具的毛病。

## 能干什么（扫的范围与后续动作）

- **扫**：整仓、指定路径（`cs scan . --path src --path tests`）、只扫一次提交差异（`cs scan . --diff origin/main`）、深度扫描（`--mode deep`，把标准扫描重复跑再去重合并）。
- **验与补**：验证候选发现是不是真能触发、生成补丁、核验别人已有的修复是否真修好。
- **政策与模型**：`cs policy .` 起草或修订 `SECURITY.md`（可加 `--path` 选组件、`--knowledge-base` 喂架构文档）；扫描与政策生成都会存下威胁模型，可用 `cs export --scan <ID> --artifact threat-model` 单独导出，不必重跑分析。
- **导出与跟踪**：SARIF、JSON、CSV 导出；发现可跟踪进 Linear、Jira、GitHub；第三方扫描器的发现可预览导入，支持的报告可发布到 Codex Security Cloud。
- **批量**：容器化的批量扫（Docker Compose， 结果与登录态在多次运行间保留），适合一串仓库排队扫。

## 产物别乱放卡

| 产物 | 放法（原仓口径） |
|---|---|
| 扫描结果 | 含源码片段与漏洞细节，是敏感材料：输出目录放在仓库外、私密保存，别顺手提交进代码仓 |
| `SECURITY.md` 草稿 | `cs policy` 把草稿存在仓库外、不动你现有文件；先审草稿与说明，改好后只把 `SECURITY.md` 本身拷到它给出的目标位置 |
| 威胁模型与架构文档 | 同样放仓库外保存；仓库根与组件目录的 `SECURITY.md` 会被扫描当作指引读入，别把内部细节写进去 |
| 已有报告说明别弄丢 | `.github/SECURITY.md`、`docs/SECURITY.md` 里常有漏洞报告指引，起草新政策时保留这些说明；但这两个位置不会自动变成全仓扫描指引 |

## 诚实标注区

- **官方文档已有西/葡语译版**：空白不在译文，在没有一页把权限边界、成本上限、扫前清凭据、三条起步路收齐的实战卡；这张补的就是这一页，如实定位、不冒充首译。
- **5 美元是估算上限不是封顶**：原仓自述进行中的请求可能超限，拿它进 CI 前先在小仓上实测一轮真实账单。
- **势头数据**：12,941 ★ / 1,011 forks、55 个 release、2,047 个 commit（2026-11-13 原仓页实测，证据 A）；今日第三方榜位第 3、一日约 +643 星为第三方统计口径（证据 B）。
- **扫描结论要复核**：它能验证候选发现、也能给补丁，但把结论当终审之前，按你自己的严重度口径再过一遍——原仓本身就支持按自家 rubric 评严重度与指派负责人。

---

## English (summary)

codex-security (12,941 stars / 1,011 forks as of 2026-11-13, Apache-2.0, 55 releases, 2,047 commits) is OpenAI's vulnerability-scanning CLI and TypeScript SDK: it scans whole repositories, selected paths or a diff, validates candidate findings, generates patches, verifies existing fixes, and ships a GitHub Action, containerized bulk scans and SARIF/JSON/CSV export. Read the permission card first: local scans do not need Codex Security Cloud — Cloud is an optional connection used to read existing Cloud findings; some cybersecurity requests and protected findings require Trusted Access for Cyber, and accounts with Daybreak access go through a separate opt-in; the CLI runs with your own operating-system permissions and does not stop to ask for approval, so move unrelated credentials out of your environment variables before scanning (CI uses `OPENAI_API_KEY` or `CODEX_API_KEY` in the scan process only). Scans also support Amazon Bedrock, OpenRouter, Fireworks AI and Cheaper Inference backends. Cost card: the official GitHub Action example sets an estimated $5 limit per scan and states that in-flight requests can exceed it; Cloud usage is billed by tokens, so read the billing hints before scanning a large repository; `cs scan --help` exposes cost limits and investigation-time options such as `--max-time 15`. Three ways in: terminal CLI (`npm install --global @openai/codex-security`, `cs login`, `cs scan .`; `cs` and `codex-security` are the same CLI, use `npx @openai/codex-security` to skip the global install), the Security plugin inside Codex, or automation via the TypeScript SDK / GitHub Action (store the key as the `CODEX_SECURITY_API_KEY` repository secret and replace the example workflow's distribution commit SHA with a reviewed one). Requirements: Node.js 22.13.0+ within 22.x, or 24.x, or 26.x; Python 3.10+, and 3.10 also needs `tomli`. Scan modes include `--path` for selected paths, `--diff origin/main` for committed changes, and `--mode deep` for repeated, deduplicated deep scans. `cs policy` drafts `SECURITY.md` outside the repository and threat models can be exported with `cs export --artifact threat-model`. Keep outputs safe: scan results contain source excerpts and vulnerability details, so store output directories outside the repository and private; policy drafts and threat models also live outside the repo, and only the reviewed `SECURITY.md` gets copied to its target. Honest note: official documentation already exists in Spanish and Portuguese — this page sells the operating rules (permissions, cost, credential hygiene, three entry routes) on one page, not a translation. Versus the Cloudflare security-audit skill covered in the companion cheatsheet, that one is an audit-methodology skill for agents; this one is a full scanner CLI plus SDK plus CI.

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

**No solo reporta vulnerabilidades: verifica si de verdad se pueden explotar y te propone el parche. Pero lee primero esta pantalla — permisos y costos — antes de escanear a ciegas.**

codex-security (12.941 estrellas / 1.011 forks a 13 de noviembre de 2026, Apache-2.0, 55 releases, 2.047 commits) es la CLI de escaneo de vulnerabilidades de OpenAI, con su SDK de TypeScript: escanea repositorios completos, rutas concretas o un diff; valida hallazgos candidatos, genera parches y verifica correcciones existentes; trae GitHub Action, escaneos masivos en contenedores y exportación SARIF/JSON/CSV. La documentación oficial ya existe en español, así que esta página no vende traducción: vende las reglas de operación en una sola hoja.

Tarjeta de permisos, fijada antes de escanear: los escaneos locales no necesitan Codex Security Cloud — Cloud es una conexión opcional que sirve para leer hallazgos ya guardados allí —; algunas solicitudes de ciberseguridad y los hallazgos protegidos exigen Trusted Access for Cyber, y las cuentas con acceso Daybreak pasan por una activación (opt-in) aparte; la CLI se ejecuta con los permisos de tu propio sistema y no se detiene a pedir aprobación, así que antes de escanear saca de las variables de entorno las credenciales que no hagan falta (en CI se usan `OPENAI_API_KEY` o `CODEX_API_KEY` solo dentro del proceso de escaneo). Los escaneos también admiten Amazon Bedrock, OpenRouter, Fireworks AI y Cheaper Inference como proveedores. Tarjeta de costos: el ejemplo oficial de GitHub Action pone un límite estimado de 5 dólares por escaneo y el propio repositorio advierte que las solicitudes en curso pueden superarlo; el uso de Cloud se factura por tokens, así que lee los avisos de facturación antes de escanear un repositorio grande; `cs scan --help` muestra las opciones de límite de costo y de tiempo de investigación, como `--max-time 15`.

Tres rutas de entrada: la CLI de terminal (`npm install --global @openai/codex-security`, luego `cs login` y `cs scan .`; `cs` y `codex-security` son la misma CLI, y `npx @openai/codex-security` evita la instalación global), el plugin Security dentro de Codex, o la automatización con el SDK de TypeScript y la GitHub Action (la clave va como secreto del repositorio `CODEX_SECURITY_API_KEY` y el SHA de distribución del workflow de ejemplo se reemplaza por uno revisado). Requisitos: Node.js 22.13.0 o superior dentro de la línea 22.x, o 24.x, o 26.x; Python 3.10 o superior, y la 3.10 necesita además `tomli`. Alcance: `--path` para rutas seleccionadas, `--diff origin/main` para cambios confirmados, `--mode deep` para escaneos profundos repetidos y deduplicados; `cs policy` redacta `SECURITY.md` fuera del repositorio y los modelos de amenazas se exportan con `cs export --artifact threat-model`. Dónde guardar cada cosa: los resultados contienen fragmentos de código fuente y detalles de vulnerabilidades, así que los directorios de salida van fuera del repositorio y en privado; los borradores de política y los modelos de amenazas también viven fuera, y solo el `SECURITY.md` ya revisado se copia a su destino. Notas honestas: el límite de 5 dólares es estimado, no una garantía de tope; y frente a la skill security-audit de Cloudflare (la hoja compañera de esta serie), aquella es una metodología de auditoría para agentes y esta es un escáner completo con CLI, SDK y CI. Esta página es una síntesis con mis propias palabras a partir de la documentación pública del proyecto, no una traducción de su README.

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

**Ele não só reporta vulnerabilidades: verifica se elas são exploráveis de verdade e propõe a correção. Mas leia primeiro esta tela — permissões e custos — antes de escanear no escuro.**

O codex-security (12.941 estrelas / 1.011 forks em 13 de novembro de 2026, Apache-2.0, 55 releases, 2.047 commits) é a CLI de varredura de vulnerabilidades da OpenAI, com o seu SDK de TypeScript: varre repositórios inteiros, caminhos concretos ou um diff; valida achados candidatos, gera correções e verifica correções existentes; traz GitHub Action, varreduras em massa em contêineres e exportação SARIF/JSON/CSV. A documentação oficial já existe em português, então esta página não vende tradução: vende as regras de operação numa folha só.

Cartão de permissões, definido antes de varrer: as varreduras locais não precisam do Codex Security Cloud — o Cloud é uma conexão opcional que serve para ler achados já salvos lá —; alguns pedidos de cibersegurança e os achados protegidos exigem Trusted Access for Cyber, e as contas com acesso Daybreak passam por uma ativação (opt-in) separada; a CLI roda com as permissões do seu próprio sistema e não para para pedir aprovação, então, antes de varrer, tire das variáveis de ambiente as credenciais desnecessárias (no CI usam-se `OPENAI_API_KEY` ou `CODEX_API_KEY` só dentro do processo de varredura). As varreduras também aceitam Amazon Bedrock, OpenRouter, Fireworks AI e Cheaper Inference como provedores. Cartão de custos: o exemplo oficial de GitHub Action coloca um limite estimado de 5 dólares por varredura, e o próprio repositório avisa que pedidos em andamento podem ultrapassá-lo; o uso do Cloud é cobrado por tokens, então leia os avisos de faturamento antes de varrer um repositório grande; `cs scan --help` mostra as opções de limite de custo e de tempo de investigação, como `--max-time 15`.

Três rotas de entrada: a CLI de terminal (`npm install --global @openai/codex-security`, depois `cs login` e `cs scan .`; `cs` e `codex-security` são a mesma CLI, e `npx @openai/codex-security` evita a instalação global), o plugin Security dentro do Codex, ou a automação com o SDK de TypeScript e a GitHub Action (a chave vai como segredo do repositório `CODEX_SECURITY_API_KEY`, e o SHA de distribuição do workflow de exemplo deve ser trocado por um revisado). Requisitos: Node.js 22.13.0 ou superior dentro da linha 22.x, ou 24.x, ou 26.x; Python 3.10 ou superior, e a 3.10 precisa também do `tomli`. Alcance: `--path` para caminhos selecionados, `--diff origin/main` para mudanças confirmadas, `--mode deep` para varreduras profundas repetidas e deduplicadas; o `cs policy` redige o `SECURITY.md` fora do repositório, e os modelos de ameaça são exportados com `cs export --artifact threat-model`. Onde guardar cada coisa: os resultados contêm trechos de código-fonte e detalhes de vulnerabilidades, então os diretórios de saída ficam fora do repositório e em privado; os rascunhos de política e os modelos de ameaça também moram fora, e só o `SECURITY.md` já revisado é copiado para o seu destino. Notas honestas: o limite de 5 dólares é estimado, não é garantia de teto; e, diante da skill security-audit da Cloudflare (a folha companheira desta série), aquela é uma metodologia de auditoria para agentes, e esta é um scanner completo com CLI, SDK e CI. Esta página é uma síntese com as minhas próprias palavras a partir da documentação pública do projeto, não uma tradução do README dele.

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [openai/codex-security](https://github.com/openai/codex-security)（2026-11-13 实测 12,941 ★ / 1,011 forks，55 个 release，2,047 个 commit，许可证 Apache-2.0；官方文档 learn.chatgpt.com/docs/security 已有西语与葡语译版）
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文，不冒充官方；权限、成本与装法以 2026-11-13 的上游 README 为准，星数以同日 GitHub 实测为准，上游迭代后请以官方文档为最终依据。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao（本仓自有内容；上游项目为 Apache-2.0）
