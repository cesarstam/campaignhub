# Prompt mestre: plataforma de automação de e-mail

Use o texto abaixo em um agente de desenvolvimento para gerar uma plataforma própria de automação de marketing. Ela deve ter recursos equivalentes de categoria a ferramentas como Mumara e ActiveCampaign, sem copiar código, marca, layout ou materiais proprietários.

## Prompt

Você é um arquiteto e engenheiro full-stack sênior. Construa uma plataforma SaaS multi-tenant de automação de marketing por e-mail chamada **CampaignHub**. O produto deve ser destinado somente a listas com consentimento verificável e cumprir LGPD, GDPR, CAN-SPAM e políticas dos provedores de envio.

### Stack obrigatória

- Frontend: Next.js 15, TypeScript, App Router, Tailwind e componentes acessíveis.
- API: NestJS ou Fastify com TypeScript, OpenAPI e validação Zod/class-validator.
- Banco: PostgreSQL 16 com migrações, índices e isolamento por `tenant_id`.
- Filas: Redis + BullMQ; jobs idempotentes, retry exponencial e dead-letter queue.
- Armazenamento: S3 compatível para imagens, exports e templates.
- Observabilidade: OpenTelemetry, logs estruturados, métricas e auditoria.
- Autenticação: OIDC/MFA, sessões seguras, RBAC e rotação de segredos.
- Deploy: Docker Compose para desenvolvimento e Kubernetes ou Docker Swarm para produção.

### Módulos funcionais

1. Tenants, usuários, equipes, RBAC, cobrança e limites por plano.
2. Contatos, campos personalizados, tags, listas, segmentos dinâmicos, importação CSV e deduplicação.
3. Consentimento: origem, timestamp, texto legal, IP/hash quando permitido, double opt-in, preferências e histórico.
4. Campanhas únicas, recorrentes e transacionais; agendamento por fuso horário.
5. Jornadas visuais com gatilhos de inscrição, evento, tag, data, condição, espera, split A/B e ações de envio/webhook.
6. Editor de e-mail com blocos responsivos, HTML sanitizado, preview desktop/mobile e testes de renderização.
7. Personalização segura (`{{first_name}}`), fallback, conteúdo condicional e links rastreáveis.
8. Formulários, páginas de preferência, confirmação e descadastro em um clique.
9. Relatórios de entrega, abertura, clique, bounce, reclamação, descadastro, conversão e receita atribuída.
10. Webhooks de provedores, eventos de produto, API pública versionada, chaves revogáveis e rate limiting.
11. Supressões globais e por tenant, gestão de bounces, complaints, bloqueios e lista de não enviar.
12. Catálogo de integrações com conectores desacoplados e tela de teste de credenciais.

### Contrato de provedor de envio

Implemente uma interface `EmailProvider` com `send`, `sendBatch`, `getStatus`, `verifyCredentials` e `parseWebhook`. Cada envio deve registrar `provider_message_id`, tenant, campanha, contato, versão do template e idempotency key.

Implemente adaptadores oficiais para Amazon SES, SendGrid, Mailgun, Postmark, Brevo, SparkPost, Resend, MailerSend, SMTP2GO, Mailjet, Elastic Email e SMTP genérico autenticado. Não embuta chaves no código. Use variáveis de ambiente/secret manager, cifre credenciais em repouso e permita um provedor primário e um secundário por tenant. Não use VPN, proxy ou rotação de IP para contornar limites ou reputação.

### Requisitos de entregabilidade e conformidade

- SPF, DKIM e DMARC documentados no onboarding.
- Domínio de envio e domínio de tracking separados do domínio principal quando apropriado.
- Cabeçalhos `List-Unsubscribe` e `List-Unsubscribe-Post`.
- Descadastro imediato e supressão permanente após reclamação.
- Limites de velocidade, warm-up explícito e bloqueio de listas sem consentimento.
- Não coletar ou vender contatos; suportar exportação e exclusão de dados.
- Proteger contra SSRF, XSS em HTML, abuso de webhooks e enumeração de contatos.

### Entregáveis

- Monorepo com `apps/web`, `apps/api`, `workers`, `packages/ui`, `packages/providers` e `infra`.
- Schema SQL/migrações, seed de demonstração e documentação OpenAPI.
- Testes unitários, integração e um fluxo E2E: importar contato com double opt-in, criar campanha, enviar por sandbox, receber webhook e aplicar descadastro.
- `docker-compose.yml`, `.env.example`, backup/restore e README de instalação.
- Checklist de segurança, privacidade, entregabilidade e operação.
- Dados de exemplo sem endereços reais e modo sandbox que não entrega e-mail.

Comece implementando um MVP funcional com contatos, consentimento, campanhas, um provedor (SES), webhooks, descadastro e dashboard. Depois adicione os demais adaptadores e jornadas sem quebrar a API.

## Infraestrutura recomendada

- Linux: Ubuntu Server 24.04 LTS, atualizado automaticamente e com acesso SSH por chave.
- Produção inicial: 4 vCPU, 8 GB RAM, 160 GB NVMe, snapshots, firewall e backups externos. Separe PostgreSQL/Redis em serviços gerenciados quando possível.
- VPN administrativa: Tailscale ou WireGuard em rede privada; permita SSH, painel, banco e Redis somente pela VPN. A VPN não deve ser usada para enviar e-mail nem para alterar IPs de entrega.
- Envio: comece com SES, Mailgun, SendGrid ou Brevo; mantenha a aplicação desacoplada para trocar de fornecedor.

## Roteiro seguro de instalação

1. Criar VPS, apontar DNS e habilitar snapshots/backup.
2. Instalar Ubuntu Server 24.04 LTS, atualizar pacotes, criar usuário sem root, desativar login SSH por senha e habilitar firewall (SSH somente pela VPN; 80/443 públicos).
3. Instalar Docker, Compose, Git e um reverse proxy com TLS automático.
4. Subir PostgreSQL, Redis e storage S3 gerenciado; gerar segredos fora do repositório.
5. Clonar o projeto, preencher `.env`, executar migrações e seed.
6. Configurar o provedor em modo sandbox, verificar credenciais e testar webhook assinado.
7. Publicar registros DNS SPF, DKIM, DMARC e CNAME de tracking; validar antes de liberar envio.
8. Ativar monitoramento, alertas de fila, bounce/complaint rate, backups e restauração testada.
9. Começar com baixo volume para contatos que deram opt-in; aumentar gradualmente conforme a reputação.

## Variáveis mínimas

`DATABASE_URL`, `REDIS_URL`, `S3_ENDPOINT`, `S3_BUCKET`, `S3_ACCESS_KEY`, `S3_SECRET_KEY`, `SESSION_SECRET`, `ENCRYPTION_KEY`, `PUBLIC_APP_URL`, `WEBHOOK_SIGNING_SECRET`, `EMAIL_PROVIDER`, `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.

Nunca commite credenciais, nunca envie para listas compradas e nunca tente contornar bloqueios de provedores.
