---
layout: post
title: "Тестовий кейс: контроль cardinality у Prometheus"
date: 2026-05-20 14:05:00 +0200
categories: [devops]
tags: [observability, prometheus, metrics]
featured: false
---

> Усі дані в статті узагальнені.

## Проблема

Зростання кількості time series погіршило запити та підвищило навантаження на сховище метрик.

## Діагностика

Виявили метрики з висококардинальними label і неактуальні серії.

## Рішення

Обмежили labels, додали relabeling і cleanup неактуальних метрик.

## Результат

Знизили cardinality та прискорили типові dashboard-запити.
