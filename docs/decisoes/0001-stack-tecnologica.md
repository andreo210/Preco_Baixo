# 0001 — Stack tecnológica

**Status:** aceita · **Data:** 2026-09-30

## Contexto
Projeto solo, com bot + coletor 24h + site com SEO, priorizando velocidade de desenvolvimento e suporte quase zero. Servidor VPS ~4 vCPU / 8 GB com Docker.

## Decisão
- **Backend/coletor em Python:** bot com **aiogram**, **API interna/admin com FastAPI** (async, Pydantic), tarefas e agendamento com **Celery + Celery Beat**, broker **Redis**.
- **Site em Next.js** (TypeScript), SSR/SSG para SEO, **consumindo a API FastAPI**.
- **PostgreSQL** para dados e histórico.
- **Docker Compose** em VPS Linux (Ubuntu LTS).

## Alternativas
- **C# / ASP.NET:** tecnologia excelente (ótima performance; Telegram.Bot, Hangfire/Quartz, EF Core, Razor/Blazor). Preterida porque, **mantendo o Next.js no site, ficam duas linguagens de qualquer jeito** — então o C# perde sua maior vantagem (unificar em Razor) e o desempate vai para o **ecossistema**, mais farto em Python para bot e integrações de afiliados (Shopee/AliExpress). Seria a escolha se o dev fosse mais produtivo em C# **ou** topasse largar o Next.js e unificar o site em Razor/Blazor.
- **TypeScript unificado** (grammY + Next.js): uma linguagem só; preterida porque Python é mais forte para coleta de dados/integrações.
- **Python full-stack** (FastAPI + Jinja/HTMX no site): SEO mais artesanal que Next.js.
- **Go:** performático, mas desenvolvimento mais lento para este escopo.

## Consequências
- Duas linguagens (Python + TS) → um pouco mais de complexidade, compensada pelo SEO forte do Next.js e pela robustez do Python na coleta.
- Celery/Redis já resolvem a frequência adaptativa e o orçamento de chamadas.
- **Fronteira definida:** o site Next.js consome a **API FastAPI** (não lê o banco direto) — resolve o antigo "A CONFIRMAR" e a questão Q-22.
