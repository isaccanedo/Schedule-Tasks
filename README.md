# 👍 Schedule-Tasks
:date: # Neste artigo, você aprenderá como agendar tarefas no Spring Boot usando a anotação @Scheduled. Você também aprenderá a usar um pool de thread personalizado para executar todas as tarefas agendadas

Neste artigo, você aprenderá como agendar tarefas no Spring Boot usando a anotação @Scheduled. Você também aprenderá a usar um pool de thread personalizado para executar todas as tarefas agendadas.

A anotação @Scheduled é adicionada a um método junto com algumas informações sobre quando executá-lo, e Spring Boot cuida do resto.

O Spring Boot usa internamente a interface TaskScheduler para agendar os métodos anotados para execução.

O objetivo deste artigo é construir um projeto simples demonstrando todos os conceitos relacionados ao agendamento de tarefas.
