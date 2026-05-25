---
layout: post
title: "Тестовый кейс: контроль cardinality в Prometheus"
date: 2026-05-20 14:05:00 +0200
categories: [devops]
tags: [observability, prometheus, metrics]
featured: false
---

> Все данные в статье обобщены.

## Проблема

Рост числа time series ухудшил запросы и увеличил нагрузку на хранилище метрик.

## Диагностика

Выявили метрики с высококардинальными label и неиспользуемые серии.

## Решение

Ограничили labels, добавили relabeling и cleanup неактуальных метрик.

## Результат

Снизили cardinality и ускорили типовые dashboard-запросы.
