# CampaignHub

Especificações para desenvolver uma plataforma própria de automação de e-mail e CRM. Este repositório contém prompts e requisitos do produto; **a aplicação ainda não foi implementada**.

## Documentos

- [Prompt completo](docs/prompt-completo-campaignhub.md): escopo de produto, arquitetura, recursos, segurança, instalação em servidor próprio, atualizações, publicação e critérios de aceite.
- [Prompt mestre resumido](docs/email-marketing-platform-prompt.md): versão menor para iniciar o desenvolvimento.

A proposta busca paridade funcional de categoria com Mumara e ActiveCampaign, sem copiar código, marca ou materiais proprietários.

## Observação sobre envio de e-mail

A aplicação pode ser self-hosted e os dados podem ficar sob controle do proprietário. A entrega de mensagens ainda exige infraestrutura de envio: um provedor oficial ou um MTA próprio, IP público, DNS correto (SPF/DKIM/DMARC e, quando aplicável, PTR) e reputação. O uso deve ser apenas com contatos que deram consentimento verificável.

## Estado

Este repositório é uma especificação pública, não um produto instalável. Não há ainda código-fonte, scripts de instalação, imagens Docker ou releases. O prompt completo define essas entregas como requisitos futuros.
