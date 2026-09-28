# code · 双色球模拟器（Java）

一个用 Java 写的双色球选号与开奖模拟程序：随机或手动选号，模拟一次开奖，按规则判断中奖等级并输出奖金。

## 规则

- 红球：1–33 中选 6 个不重复号码
- 蓝球：1–16 中选 1 个号码

| 奖级 | 中奖条件（红 + 蓝） | 程序内奖金 |
| --- | --- | --- |
| 一等奖 | 6 + 1 | 5,000,000 元 |
| 二等奖 | 6 + 0 | 500,000 元 |
| 三等奖 | 5 + 1 | 3,000 元 |
| 四等奖 | 5 + 0 或 4 + 1 | 200 元 |
| 五等奖 | 4 + 0 或 3 + 1 | 10 元 |
| 六等奖 | 2 + 1、1 + 1 或 0 + 1 | 5 元 |

一、二等奖在真实开奖中是浮动奖金，程序里用固定数值代替，仅作演示。

## 运行

需要 JDK 11 及以上。把下面的代码保存为 `DoubleColorBall.java`，然后：

```bash
javac DoubleColorBall.java
java DoubleColorBall
```

输出示例：

```
您的投注：红球[3, 8, 15, 21, 27, 30] + 蓝球[9]
开奖号码：红球[1, 8, 12, 21, 26, 33] + 蓝球[9]
恭喜！您中了5元
```

默认是机选。想手动选号，把 `main` 里的两行 `manualSelectRed()` / `manualSelectBlue()` 取消注释，并注释掉上面两行机选即可。

## 源码

```java
import java.util.*;
import java.util.stream.Collectors;

public class DoubleColorBall {
    // 中奖规则配置（单位：元）
    private static final Map<Integer, Integer> PRIZE_RULES = new HashMap<>() {{
        put(1, 5_000_000);  // 一等奖：6+1
        put(2, 500_000);    // 二等奖：6+0
        put(3, 3000);       // 三等奖：5+1
        put(4, 200);        // 四等奖：5+0或4+1
        put(5, 10);         // 五等奖：4+0或3+1
        put(6, 5);          // 六等奖：2+1、1+1、0+1
    }};

    public static void main(String[] args) {
        // 用户投注（可切换手动/自动模式）
        List<Integer> userRed = generateRedBalls();  // 自动生成红球
        int userBlue = generateBlueBall();           // 自动生成蓝球
        // List<Integer> userRed = manualSelectRed(); // 手动输入红球
        // int userBlue = manualSelectBlue();        // 手动输入蓝球

        // 模拟开奖
        List<Integer> prizeRed = generateRedBalls();
        int prizeBlue = generateBlueBall();
        
        // 输出结果
        System.out.println("您的投注：红球" + userRed + " + 蓝球[" + userBlue + "]");
        System.out.println("开奖号码：红球" + prizeRed + " + 蓝球[" + prizeBlue + "]");
        
        // 计算中奖等级
        int prizeLevel = checkPrize(userRed, userBlue, prizeRed, prizeBlue);
        System.out.println(prizeLevel > 0 ? 
            "恭喜！您中了" + PRIZE_RULES.get(prizeLevel) + "元" : "未中奖");
    }

    // 生成红球（1-33选6不重复）
    private static List<Integer> generateRedBalls() {
        Set<Integer> redBalls = new TreeSet<>();
        Random rand = new Random();
        while (redBalls.size() < 6) {
            redBalls.add(rand.nextInt(33) + 1);
        }
        return new ArrayList<>(redBalls);
    }

    // 生成蓝球（1-16选1）
    private static int generateBlueBall() {
        return new Random().nextInt(16) + 1;
    }

    // 中奖等级判断（核心逻辑）
    private static int checkPrize(List<Integer> userRed, int userBlue, 
                                 List<Integer> prizeRed, int prizeBlue) {
        // 计算红球匹配数
        Set<Integer> userSet = new HashSet<>(userRed);
        Set<Integer> prizeSet = new HashSet<>(prizeRed);
        userSet.retainAll(prizeSet); // 取交集
        int redMatches = userSet.size();
        
        // 蓝球是否匹配
        boolean blueMatch = (userBlue == prizeBlue);

        // 判断中奖等级
        if (redMatches == 6 && blueMatch) return 1;
        if (redMatches == 6) return 2;
        if (redMatches == 5 && blueMatch) return 3;
        if (redMatches == 5 || (redMatches == 4 && blueMatch)) return 4;
        if (redMatches == 4 || (redMatches == 3 && blueMatch)) return 5;
        if (blueMatch) return 6;
        return 0;
    }

    // 手动输入红球（带校验）
    private static List<Integer> manualSelectRed() {
        Scanner scanner = new Scanner(System.in);
        Set<Integer> reds = new TreeSet<>();
        System.out.println("请输入6个红球（1-33，空格分隔）：");
        while (reds.size() < 6) {
            int num = scanner.nextInt();
            if (num < 1 || num > 33) {
                System.out.println("无效号码，请重新输入（1-33）：");
            } else if (!reds.add(num)) {
                System.out.println("号码重复，请重新输入：");
            }
        }
        return new ArrayList<>(reds);
    }

    // 手动输入蓝球（带校验）
    private static int manualSelectBlue() {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入蓝球（1-16）：");
        int blue;
        do {
            blue = scanner.nextInt();
        } while (blue < 1 || blue > 16);
        return blue;
    }
}
```
