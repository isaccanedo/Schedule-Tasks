# 👍 Schedule-Tasks
:date: # Neste artigo, você aprenderá como agendar tarefas no Spring Boot usando a anotação @Scheduled. Você também aprenderá a usar um pool de thread personalizado para executar todas as tarefas agendadas

# 1. Visão geral

A anotação @Scheduled é adicionada a um método junto com algumas informações sobre quando executá-lo, e Spring Boot cuida do resto.

O Spring Boot usa internamente a interface TaskScheduler para agendar os métodos anotados para execução.

O objetivo deste artigo é construir um projeto simples demonstrando todos os conceitos relacionados ao agendamento de tarefas.

# 2. Habilitar agendamento
Você pode ativar o agendamento simplesmente adicionando a anotação @EnableScheduling à classe de aplicativo principal ou uma das classes de configuração.

# 3. Agendamento de tarefas
Agendar uma tarefa com Spring Boot é tão simples quanto anotar um método com anotação @Scheduled e fornecer alguns parâmetros que serão usados para decidir a hora em que a tarefa será executada.

Antes de adicionar tarefas, vamos primeiro criar o contêiner para todas as tarefas agendadas. Crie uma nova classe chamada ScheduledTasks dentro do pacote 

A classe contém quatro métodos vazios. Veremos a implementação de todos os métodos, um por um.

Todos os métodos programados devem seguir os dois critérios a seguir -

- O método deve ter um tipo de retorno nulo;
- O método não deve aceitar nenhum argumento.

# 4. Agendamento de uma tarefa com taxa fixa
Você pode agendar um método para ser executado em um intervalo fixo usando o parâmetro fixedRate na anotação @Scheduled. No exemplo a seguir, o método anotado será executado a cada 2 segundos.

A tarefa fixedRate é chamada no intervalo especificado, mesmo se a chamada anterior da tarefa não for concluída.

# 5. Agendamento de uma tarefa com atraso fixo
Você pode executar uma tarefa com um atraso fixo entre a conclusão da última invocação e o início da próxima, usando o parâmetro fixedDelay.

O parâmetro fixedDelay conta o atraso após a conclusão da última chamada.

```
@Scheduled(fixedDelay = 2000)
public void scheduleTaskWithFixedDelay() {
    logger.info("Fixed Delay Task :: Execution Time - {}", dateTimeFormatter.format(LocalDateTime.now()));
    try {
        TimeUnit.SECONDS.sleep(5);
    } catch (InterruptedException ex) {
        logger.error("Ran into an error {}", ex);
        throw new IllegalStateException(ex);
    }
}
```

Como a tarefa em si leva 5 segundos para ser concluída e especificamos um atraso de 2 segundos entre a conclusão da última invocação e o início da próxima, haverá um atraso de 7 segundos entre cada invocação

```
# Sample Output
Fixed Delay Task :: Execution Time - 10:30:01
Fixed Delay Task :: Execution Time - 10:30:08
Fixed Delay Task :: Execution Time - 10:30:15
....
....
```

# 6. Agendamento de uma tarefa com taxa fixa e atraso inicial
Você pode usar o parâmetro initialDelay com fixedRate e fixedDelay para atrasar a primeira execução da tarefa com o número especificado de milissegundos.

No exemplo a seguir, a primeira execução da tarefa será atrasada em 5 segundos e então será executada normalmente em um intervalo fixo de 2 segundos

```
@Scheduled(fixedRate = 2000, initialDelay = 5000)
public void scheduleTaskWithInitialDelay() {
    logger.info("Fixed Rate Task with Initial Delay :: Execution Time - {}", dateTimeFormatter.format(LocalDateTime.now()));
}
```

```
# Sample output (Server Started at 10:48:46)
Fixed Rate Task with Initial Delay :: Execution Time - 10:48:51
Fixed Rate Task with Initial Delay :: Execution Time - 10:48:53
Fixed Rate Task with Initial Delay :: Execution Time - 10:48:55
....
....
```

# 7. Agendando uma tarefa usando a expressão Cron
Se os parâmetros simples acima não podem atender às suas necessidades, então você pode usar expressões cron para agendar a execução de suas tarefas.

No exemplo a seguir, programei a tarefa para ser executada a cada minuto

```
@Scheduled(cron = "0 * * * * ?")
public void scheduleTaskWithCronExpression() {
    logger.info("Cron Task :: Execution Time - {}", dateTimeFormatter.format(LocalDateTime.now()));
}
```

```
# Sample Output
Cron Task :: Execution Time - 11:03:00
Cron Task :: Execution Time - 11:04:00
Cron Task :: Execution Time - 11:05:00
```

Executando tarefas @Scheduled em um pool de threads personalizadas
Por padrão, todas as tarefas @Scheduled são executadas em um pool de threads padrão de tamanho um criado pelo Spring.

Você pode verificar isso registrando o nome do segmento atual em todos os métodos

```
logger.info("Current Thread : {}", Thread.currentThread().getName());
```

Todos os métodos imprimirão o seguinte

```
Current Thread : pool-1-thread-1
```

Mas hey, você pode criar seu próprio pool de threads e configurar o Spring para usar esse pool de threads para executar todas as tarefas agendadas.

Crie uma nova configuração de pacote dentro de com.example.schedulerdemo e, em seguida, crie uma nova classe chamada SchedulerConfig dentro do pacote de configuração com o seguinte conteúdo.

```
package com.example.schedulerdemo.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;

@Configuration
public class SchedulerConfig implements SchedulingConfigurer {
    private final int POOL_SIZE = 10;

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();

        threadPoolTaskScheduler.setPoolSize(POOL_SIZE);
        threadPoolTaskScheduler.setThreadNamePrefix("my-scheduled-task-pool-");
        threadPoolTaskScheduler.initialize();

        scheduledTaskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
    }
}
```

Isso é tudo que você precisa fazer para configurar o Spring para usar seu próprio pool de threads em vez do padrão.

Se você registrar o nome do thread atual nos métodos programados agora, você receberá a saída como esta:

```
Current Thread : my-scheduled-task-pool-1
Current Thread : my-scheduled-task-pool-2

# etc...
```

# 8. Conclusão
Neste artigo, você aprendeu como agendar tarefas no Spring Boot usando a anotação @Scheduled. Você também aprendeu como usar um pool de threads customizado para executar essas tarefas.

A abstração Scheduling fornecida pelo Spring Boot funciona muito bem para casos de uso simples. Mas se você tiver casos de uso mais avançados, como trabalhos persistentes, clustering, adição dinâmica e acionamento de novos trabalhos, verifique o seguinte artigo -

Exemplo de agendador de Spring Boot Quartz
