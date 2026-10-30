# Prompt completo: CampaignHub self-hosted

Copie o prompt abaixo para um agente de desenvolvimento. Ele deve gerar uma plataforma própria de automação de e-mail e CRM, com recursos equivalentes de categoria a Mumara e ActiveCampaign, sem copiar código, marca, layout, textos ou ativos proprietários.

## Prompt para o agente

Você é uma equipe sênior de arquitetura, produto, segurança, entregabilidade e engenharia full-stack. Construa uma plataforma chamada **CampaignHub**, controlada pelo proprietário, self-hosted e multi-tenant, para marketing autorizado por e-mail. O sistema deve funcionar em domínio próprio e ser instalável em Ubuntu Server 24.04 LTS.

O produto deve atender LGPD, GDPR, CAN-SPAM, CASL e políticas dos provedores. Nunca implemente disparo para listas compradas, scraping de contatos, spam, falsificação de remetente, rotação de IP, evasão de bloqueios ou bypass de limites.

### Objetivo de independência

Torne a aplicação, banco, filas, templates, analytics, consentimento, auditoria, usuários e configurações controlados pelo proprietário. Suporte dois modos de entrega:

1. **Modo recomendado:** conectores para provedores oficiais de e-mail.
2. **Modo self-hosted:** MTA próprio opcional, com Postfix/Haraka/Postal ou equivalente, somente para contatos com opt-in, com controle de fila, bounce, complaint, SPF, DKIM, DMARC, PTR, TLS, warm-up e reputação.

Explique no README que nenhum software consegue entregar e-mail diretamente na caixa de entrada sem um MTA, IP público, DNS correto e reputação. A plataforma não deve esconder essa dependência.

## Stack moderna

- Frontend: Next.js atual, React, TypeScript, App Router, Tailwind, componentes acessíveis e editor visual baseado em blocos.
- Backend: NestJS ou Fastify com TypeScript, REST versionado, OpenAPI, validação Zod/class-validator e WebSocket para status de jobs.
- Banco principal: PostgreSQL 16 ou superior, migrações, índices, particionamento de eventos e isolamento rigoroso por `tenant_id`.
- Filas: Redis 7 ou superior + BullMQ, idempotência, retry exponencial, backoff, prioridade, rate limit e dead-letter queue.
- Armazenamento: S3 compatível ou MinIO para imagens, anexos, exports e versões de templates.
- Busca opcional: OpenSearch/Meilisearch para contatos e auditoria.
- Autenticação: OIDC, MFA/WebAuthn, recuperação segura, sessões rotativas, RBAC e suporte opcional a Keycloak.
- Segurança: secret manager, criptografia em repouso, TLS, CSP, CSRF, SSRF protection, sanitização de HTML, rate limiting e auditoria imutável.
- Observabilidade: OpenTelemetry, logs estruturados, métricas Prometheus, dashboards Grafana, alertas e tracing.
- Infraestrutura: Docker Compose para desenvolvimento, Terraform para infraestrutura e Kubernetes ou Docker Swarm para produção.
- CI/CD: lint, typecheck, testes, SAST, dependency scan, migrations e deploy com rollback.

## Módulos obrigatórios

### 1. Contas e organizações

- Multi-tenant com organizações, equipes, usuários, papéis e permissões.
- Planos, limites, consumo, faturamento configurável e feature flags.
- Preferências de idioma, fuso horário, marca, domínio e remetente.
- API keys por organização, escopos, expiração, revogação e auditoria.

### 2. Contatos e consentimento

- Contatos, empresas, campos personalizados, tags, listas e segmentos dinâmicos.
- Importação CSV/XLSX com preview, mapeamento de campos, deduplicação e rollback.
- Eventos de produto, pontuação, origem, UTM e histórico de atividades.
- Consentimento com origem, finalidade, versão do texto legal, timestamp, IP/hash quando permitido e prova de double opt-in.
- Preferências por categoria, centro de preferências, descadastro em um clique e exportação/exclusão LGPD.
- Supressão global e por organização para bounce permanente, complaint, descadastro e bloqueio manual.

### 3. Campanhas

- Campanhas únicas, recorrentes, transacionais e baseadas em evento.
- Agendamento por fuso horário, janela de envio, limite diário e throttling.
- Editor drag-and-drop por blocos, HTML sanitizado, preview desktop/mobile e texto alternativo.
- Variáveis seguras como `{{first_name}}`, fallback, conteúdo condicional e personalização por segmento.
- Teste de envio, aprovação em etapas, versionamento, rascunho, revisão e rollback.
- A/B testing de assunto, remetente, conteúdo, horário e grupo de controle.

### 4. Jornadas de automação

- Builder visual com gatilhos: inscrição, formulário, tag, evento, compra, webhook, data e inatividade.
- Condições, espera, horário, split A/B, limites de frequência, metas, saída e reentrada controlada.
- Ações: e-mail, webhook, atualização de campo, tag, segmento, notificação e chamada HTTP assinada.
- Execução durável, retomável e idempotente, com histórico de cada contato.

### 5. Formulários e páginas

- Formulários incorporáveis, páginas de captura, confirmação e centro de preferências.
- Double opt-in, proteção anti-abuso, captcha/Turnstile opcional, validação e rate limiting.
- Domínio personalizado, HTTPS, favicon, SEO básico e páginas de descadastro acessíveis.

### 6. Analytics e entregabilidade

- Entrega, abertura, clique, bounce, complaint, descadastro, conversão e receita atribuída.
- Relatórios por campanha, jornada, segmento, domínio, provedor e período.
- Funil, cohort, retenção, heatmap de cliques e comparação A/B.
- Monitor de reputação, bounce/complaint rate, fila, latência, bloqueios e falhas de webhook.
- Links de tracking assinados, proteção contra open-tracking enganoso e opção de desativar tracking.
- Cabeçalhos `List-Unsubscribe` e `List-Unsubscribe-Post`.

### 7. Integrações

Implemente uma interface estável `EmailProvider`:

```ts
interface EmailProvider {
  verifyCredentials(): Promise<ProviderHealth>;
  send(message: OutboundMessage): Promise<SendResult>;
  sendBatch(messages: OutboundMessage[]): Promise<SendResult[]>;
  getStatus(providerMessageId: string): Promise<DeliveryStatus>;
  parseWebhook(request: SignedWebhookRequest): ProviderEvent[];
}
```

Crie adaptadores oficiais e testáveis para:

- Amazon SES
- SendGrid
- Mailgun
- Postmark
- Brevo
- SparkPost
- Resend
- MailerSend
- SMTP2GO
- Mailjet
- Elastic Email
- SMTP autenticado genérico
- MTA próprio via SMTP/HTTP interno

Não afirme que a lista é literalmente todo o mercado. Crie um registro de plugins para novos conectores sem alterar o núcleo.

Cada envio deve registrar `tenant_id`, contato, campanha, jornada, template version, provider, `provider_message_id`, idempotency key, timestamps, status e motivo de falha.

Suporte provedor primário/secundário, failover somente para erros elegíveis, circuit breaker, limites por provedor e reconciliação por webhook. Nunca use failover para contornar bloqueios de abuso.

## Domínio próprio e DNS

Crie onboarding guiado para:

- `app.exemplo.com` para o painel;
- `track.exemplo.com` para links rastreáveis;
- `mail.exemplo.com` para o domínio de envio, se aplicável;
- SPF;
- DKIM;
- DMARC com modo de observação inicial e posterior endurecimento;
- MX/Return-Path para bounces;
- PTR/reverse DNS quando o MTA for próprio;
- TLS e renovação automática de certificados.

Valide os registros antes de permitir envio de produção. Mostre instruções exatas e status de propagação.

## Instalação e operação

Gere documentação testada para Ubuntu Server 24.04 LTS:

1. Criar VPS, snapshots, backup externo e usuário sem root.
2. Instalar atualizações, Docker Engine, Compose, Git, firewall e reverse proxy.
3. Configurar Tailscale ou WireGuard apenas para administração.
4. Subir PostgreSQL, Redis, MinIO/S3, aplicação, workers e monitoramento.
5. Gerar segredos fora do Git e preencher `.env` a partir de `.env.example`.
6. Executar migrations, seed de demonstração e health checks.
7. Configurar domínio, HTTPS, SPF, DKIM, DMARC, PTR e tracking.
8. Validar um provedor em sandbox e um fluxo de webhook assinado.
9. Liberar produção gradualmente somente para contatos com consentimento.
10. Testar backup, restauração, rotação de segredo, rollback e recuperação de fila.

Nunca documente como prática válida executar scripts desconhecidos com `curl | sh`. Para instaladores externos, baixe para arquivo, confira assinatura/checksum publicado pelo fornecedor, leia o script e só então execute com o privilégio mínimo necessário.

## Variáveis de ambiente mínimas

`DATABASE_URL`, `REDIS_URL`, `S3_ENDPOINT`, `S3_BUCKET`, `S3_ACCESS_KEY`, `S3_SECRET_KEY`, `SESSION_SECRET`, `ENCRYPTION_KEY`, `PUBLIC_APP_URL`, `TRACKING_BASE_URL`, `WEBHOOK_SIGNING_SECRET`, `EMAIL_PROVIDER`, `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.

Credenciais devem ser cifradas em repouso, mascaradas em logs, rotacionáveis e nunca commitadas.

## API pública

Implemente `/api/v1` com OpenAPI para organizações e usuários, contatos, listas, tags, segmentos, consentimentos, preferências, templates, campanhas, jornadas, eventos, envio transacional, webhooks, analytics, exports, exclusão e auditoria.

Use paginação, filtros, idempotência, ETags, rate limits, escopos e respostas de erro consistentes.

## Testes e critérios de aceite

Entregue:

- monorepo com `apps/web`, `apps/api`, `workers`, `packages/ui`, `packages/providers`, `packages/config` e `infra`;
- migrations, seed e dados fictícios sem e-mails reais;
- Docker Compose funcional;
- `.env.example`, README e playbook de operações;
- testes unitários, integração, contrato e E2E;
- fluxo E2E: importar contato, registrar consentimento, double opt-in, criar campanha, enviar em sandbox, processar webhook, medir evento e descadastrar;
- teste de isolamento entre tenants;
- teste de segurança para XSS, SSRF, replay de webhook e enumeração de contatos;
- backup/restore automatizado;
- health/readiness/liveness checks;
- modo sandbox que nunca entrega e-mail real;
- checklist de LGPD, segurança, entregabilidade e resposta a incidentes.

## Ordem de implementação

1. Fundamentos: monorepo, autenticação, tenant, banco, auditoria e Docker.
2. Contatos, consentimento, supressões e descadastro.
3. Templates, campanhas, modo sandbox e dashboard.
4. SES e SMTP genérico com webhooks.
5. Filas, workers, rate limiting e analytics.
6. Jornadas visuais e eventos.
7. Demais provedores e failover controlado.
8. MTA próprio opcional e documentação de reputação.
9. Observabilidade, hardening, backup e produção.

Não declare o sistema pronto enquanto migrations, testes E2E, descadastro, supressão, isolamento de tenant e verificação de webhook não estiverem funcionando.

## Resultado esperado

Entregue código executável, documentação de instalação, arquitetura, modelo de dados, OpenAPI, testes, exemplos de DNS, matriz de provedores, threat model e checklist de lançamento. O sistema deve permitir ao proprietário controlar a aplicação e os dados, mas deve declarar honestamente os requisitos externos de IP, DNS, MTA e reputação necessários para entregar mensagens na Internet.

## Publicação obrigatória no GitHub

Prepare o projeto para publicação em um repositório GitHub privado ou público, sem incluir credenciais, dados pessoais, dumps, chaves ou arquivos `.env`.

Entregue:

- `README.md` em português, com visão geral, requisitos, instalação rápida, instalação manual, configuração DNS, atualização, backup, restauração e solução de problemas;
- `LICENSE` escolhida explicitamente pelo proprietário; se nenhuma licença for definida, mantenha o repositório privado e marque o código como proprietário;
- `SECURITY.md`, política de divulgação responsável e versões suportadas;
- `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, changelog e templates de issue/pull request;
- `.gitignore`, `.dockerignore`, `.editorconfig`, `.env.example` sem segredos e arquivo de exemplo para produção;
- proteção da branch principal, revisão obrigatória, status checks e Dependabot/Renovate;
- releases semânticas, tags assinadas quando possível, notas de versão e checksums SHA-256 dos artefatos;
- SBOM em CycloneDX ou SPDX, scan de dependências, secrets scan, SAST e scan das imagens Docker;
- imagens versionadas publicadas no GitHub Container Registry, nunca apenas com a tag `latest`;
- documentação para clonar por HTTPS e SSH e para criar um fork privado;
- instruções para alterar nome, logotipo, domínio e identidade visual sem editar o núcleo.

Crie workflows em `.github/workflows`:

1. `ci.yml`: lint, formatação, typecheck, testes unitários, integração e build.
2. `security.yml`: secrets scan, dependency audit, SAST, SBOM e scan de container.
3. `e2e.yml`: sobe o Docker Compose em ambiente descartável e executa o fluxo E2E.
4. `release.yml`: cria release versionada, changelog, imagens no GHCR e checksums.
5. `deploy.yml`: deployment manual com environment protegido, aprovação e rollback.

Nenhum workflow pode imprimir segredos nos logs. Use GitHub Environments, OIDC quando disponível e permissões mínimas no `GITHUB_TOKEN`.

Inclua no README os comandos iniciais:

```bash
git clone https://github.com/SEU_USUARIO/campaignhub.git
cd campaignhub
cp .env.example .env
./scripts/preflight.sh
./scripts/install.sh
```

Explique como criar o repositório:

```bash
git init
git add .
git commit -m "feat: initial CampaignHub release"
git branch -M main
git remote add origin git@github.com:SEU_USUARIO/campaignhub.git
git push -u origin main
```

O agente deve solicitar ou usar placeholders para `SEU_USUARIO` e URL do repositório. Nunca invente uma conta GitHub nem publique sem autorização explícita.

## Instalação fácil e reproduzível

Crie uma instalação assistida para Ubuntu Server 24.04 LTS x86_64/ARM64. Ela deve ser idempotente: executar novamente não pode apagar dados nem duplicar configurações.

Forneça estes scripts versionados:

- `scripts/preflight.sh`: verifica sistema operacional, arquitetura, CPU, RAM, disco, portas, DNS, relógio/NTP, usuário, permissões e conectividade;
- `scripts/bootstrap-ubuntu.sh`: instala somente dependências do sistema após confirmação;
- `scripts/install.sh`: cria diretórios, valida `.env`, gera segredos ausentes, baixa ou constrói imagens, inicia serviços, executa migrations e health checks;
- `scripts/configure-domain.sh`: configura domínio, reverse proxy e certificado TLS;
- `scripts/configure-provider.sh`: assistente para SES, SMTP ou outro conector, com teste em sandbox;
- `scripts/update.sh`: backup, validação, atualização, migrations compatíveis, health check e rollback automático em falha;
- `scripts/backup.sh` e `scripts/restore.sh`: backup cifrado e restauração testável;
- `scripts/doctor.sh`: diagnóstico sem revelar segredos;
- `scripts/uninstall.sh`: remove somente containers e aplicação após confirmação, preservando backups por padrão.

Todos os scripts devem usar modo estrito, mensagens claras, códigos de saída, logs sem segredos e confirmação antes de ações destrutivas. Eles devem aceitar `--help`, `--non-interactive` e, quando aplicável, `--dry-run`.

Não use `curl URL | sh`. Para uma release:

```bash
curl --fail --location --proto '=https' --tlsv1.2 \
  -o campaignhub-installer.tar.gz \
  https://github.com/SEU_USUARIO/campaignhub/releases/download/vX.Y.Z/campaignhub-installer.tar.gz

curl --fail --location --proto '=https' --tlsv1.2 \
  -o SHA256SUMS \
  https://github.com/SEU_USUARIO/campaignhub/releases/download/vX.Y.Z/SHA256SUMS

sha256sum --check SHA256SUMS
tar -xzf campaignhub-installer.tar.gz
cd campaignhub-installer
less install.sh
sudo ./install.sh
```

Documente também a instalação manual completa para auditoria e recuperação caso o instalador assistido falhe.

## Componentes necessários

O instalador deve verificar e instalar/configurar, sem substituir configurações existentes silenciosamente:

- Ubuntu Server 24.04 LTS atualizado;
- ca-certificates, curl, Git, jq, OpenSSL, unzip, cron/systemd e utilitários de backup;
- Docker Engine oficial, Docker Compose v2 e Buildx;
- firewall UFW ou nftables;
- Caddy, Traefik ou Nginx como reverse proxy com TLS automático;
- PostgreSQL 16;
- Redis 7;
- MinIO local ou credenciais de S3 compatível;
- aplicação web, API e workers;
- migrations e seed opcional;
- Prometheus, Grafana e coleta de logs, como perfil opcional de observabilidade;
- Tailscale ou WireGuard como perfil opcional de administração;
- ClamAV somente se anexos/upload exigirem antivírus;
- Restic ou ferramenta equivalente para backup cifrado externo.

Prefira containers para PostgreSQL, Redis, MinIO e serviços da aplicação na instalação simples. Para produção de maior escala, documente a migração para serviços gerenciados ou cluster separado.

Defina no `docker-compose.yml`:

- health checks e dependências baseadas em saúde;
- volumes nomeados persistentes;
- redes internas separadas;
- limites de recursos;
- reinício controlado;
- usuário não root sempre que possível;
- arquivos somente leitura e capabilities mínimas;
- profiles `core`, `observability`, `mta` e `development`;
- nenhuma porta de banco, Redis ou MinIO publicada na Internet;
- apenas HTTP/HTTPS expostos publicamente.

## Assistente de configuração inicial

Após a instalação, execute um wizard em `https://app.DOMINIO` ou CLI para:

1. Criar a primeira conta proprietária com MFA.
2. Informar nome da organização, idioma e fuso horário.
3. Configurar `app.DOMINIO`, `track.DOMINIO` e domínio de envio.
4. Exibir e validar SPF, DKIM, DMARC, Return-Path e PTR quando necessário.
5. Escolher modo sandbox, provedor externo ou MTA próprio.
6. Testar credenciais e webhook.
7. Criar lista, formulário, template e campanha de demonstração.
8. Enviar somente para um endereço verificado do proprietário.
9. Confirmar backup e alertas antes de habilitar produção.

O modo produção deve permanecer bloqueado enquanto domínio, descadastro, supressões, autenticação do remetente e webhooks críticos não estiverem validados.

## Atualização e manutenção

Use versionamento semântico e migrations compatíveis com rollback. O comando padrão deve ser:

```bash
cd /opt/campaignhub
sudo ./scripts/update.sh --version vX.Y.Z
```

O processo de atualização deve:

1. Verificar espaço, versão atual, release, checksum/assinatura e compatibilidade.
2. Criar backup do banco, configurações, volumes críticos e versão anterior.
3. Baixar imagens com tags imutáveis.
4. Executar migrations com estratégia expand/contract quando necessário.
5. Atualizar API, workers e frontend na ordem segura.
6. Executar health checks, smoke tests e teste de fila.
7. Reverter containers e migrations reversíveis se a validação falhar.
8. Registrar o resultado na auditoria e preservar logs de manutenção.

Inclua comandos documentados:

```bash
./scripts/doctor.sh
./scripts/backup.sh
./scripts/update.sh --check
./scripts/update.sh --version vX.Y.Z
./scripts/restore.sh --list
docker compose ps
docker compose logs --tail=200 api worker web
```

Crie uma política de atualização para patch, minor e major, incluindo janela de manutenção, breaking changes, deprecações e tempo mínimo de suporte.

## Manual para usuário iniciante

O README deve conter uma seção “Instalação em 15 minutos” com telas ou exemplos, linguagem simples e estes dados solicitados antes de começar:

- IP público do VPS;
- domínio e acesso ao DNS;
- e-mail administrativo;
- provedor de envio escolhido ou decisão por modo sandbox;
- credenciais do S3/provedor, quando aplicável;
- destino do backup;
- senha forte ou geração automática.

Inclua uma tabela de portas:

| Porta | Exposição | Uso |
|---|---|---|
| 22 | VPN/IP administrativo | SSH |
| 80 | Pública | Redirecionamento e desafio TLS |
| 443 | Pública | Painel, API, tracking e webhooks |
| 25/465/587 | Conforme o modo MTA | SMTP; não abrir sem necessidade |
| 5432 | Interna | PostgreSQL |
| 6379 | Interna | Redis |
| 9000/9001 | Interna/VPN | MinIO |

Inclua troubleshooting para DNS não propagado, certificado, porta ocupada, migrations, falta de memória, fila parada, webhook inválido, credenciais recusadas, bounce elevado e restauração de backup.

Ao finalizar a implementação, execute a instalação do zero em uma VM limpa suportada e uma atualização da versão anterior. Registre evidências dos health checks e não considere a documentação pronta sem validar ambos os fluxos.
