## 基本用法
将@Scheduled注解应用到你希望定时执行的方法上。通常，你需要在Spring的配置类中启用定时任务的支持，通常是通过@EnableScheduling注解来实现。

## 属性
- fixedRate: 指定上一次执行完毕到下一次执行开始之间的固定间隔时间（单位是毫秒）。例如，fixedRate = 5000 表示每5秒执行一次。

- fixedDelay: 指定上一次执行完毕到下一次执行开始之前的延迟时间（单位是毫秒）。与fixedRate不同，它考虑了方法执行所需的时间，因此两次执行的实际间隔可能并不固定。

- initialDelay: 首次执行前的延迟时间（单位是毫秒）。可以与fixedRate或fixedDelay一起使用，以设定首次执行的偏移时间。

- cron: 使用Cron表达式来指定执行时间。Cron表达式提供了非常灵活的时间控制，可以精确到秒，支持各种复杂的定时规则，如每天的特定时间执行、每周的某几天执行等。

## 实例
```
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ScheduledTasks {

    @Scheduled(fixedRate = 5000)
    public void fixedRateTask() {
        System.out.println("Fixed Rate Task - " + new Date());
    }

    @Scheduled(fixedDelay = 5000)
    public void fixedDelayTask() {
        System.out.println("Fixed Delay Task - " + new Date());
    }

    @Scheduled(initialDelay = 1000, fixedRate = 10000)
    public void initialDelayTask() {
        System.out.println("Initial Delay Task - " + new Date());
    }

    @Scheduled(cron = "0 0 12 * * ?") // 每天中午12点执行
    public void cronTask() {
        System.out.println("Cron Task - " + new Date());
    }
}
```
### cron表达式
@Scheduled(cron = "0 0/1 * * * *") 使用的是cron表达式来定义定时任务的执行时间。这个cron表达式的含义分解如下：

0: 秒，表示每分钟的第0秒执行。
0/1: 分，这里的0/1意味着从第0分钟开始，每隔1分钟执行一次。在cron表达式中，这种写法等同于简单地写分钟的范围（比如*），但由于指定了步进（/1），明确表达了每隔一分钟的意图。
*: 小时、日、月、周，分别表示任意小时、任意一天的任意时间、任意月份、一周中的任意一天。
综上所述，这个cron表达式定义的定时任务将会每分钟执行一次，因为秒设置为0，分钟设置为每分钟执行（通过0/1表达每隔1分钟，实际上就是每分钟）。所以，这个任务将在每天的每一分钟的第0秒执行。