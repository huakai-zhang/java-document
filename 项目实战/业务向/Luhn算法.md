```java
/**
 * 使用 luhn 算法精准校验银行卡号
 * 校验过程：
 * 1、从卡号最后一位数字开始，逆向将奇数位(1、3、5等等)相加。
 * 2、从卡号最后一位数字开始，逆向将偶数位数字，先乘以2（如果乘积为两位数，将个位十位数字相加，即将其减去9），再求和。
 * 3、将奇数位总和加上偶数位总和，结果应该可以被10整除。
 */
public class LuhnUtil {

    /**
     * 校验银行卡卡号
     */
    public static boolean checkBankCard(String bankCard) {
        if (bankCard == null || bankCard.trim().length() == 0) {
            return false;
        }
        if (bankCard.length() < 16 || bankCard.length() > 19) {
            return false;
        }
        // 根据Luhn法则得到校验位
        char bit = getBankCardCheckCode(bankCard.substring(0, bankCard.length() - 1));
        if (bit == 'N') {
            return false;
        }
        // 和银行卡的校验位(最后一位)比较
        return bankCard.charAt(bankCard.length() - 1) == bit;
    }

    /**
     * 从不含校验位的银行卡卡号采用 Luhm 校验算法获得校验位
     */
    public static char getBankCardCheckCode(String nonCheckCodeBankCard) {
        if (nonCheckCodeBankCard == null || nonCheckCodeBankCard.trim().length() == 0
                || !nonCheckCodeBankCard.matches("\\d+")) {
            //如果传的不是数据返回N
            return 'N';
        }
        char[] chs = nonCheckCodeBankCard.trim().toCharArray();
        int luhmSum = 0;
        // 注意是从下标为0出开始
        for (int i = chs.length - 1, j = 0; i >= 0; i--, j++) {
            int k = chs[i] - '0';
            // 是偶数位数字做处理先乘以2（如果乘积为两位数，则将其减去9或个位数十位相加的和），再求和
            // 不是则不做处理
            if (j % 2 == 0) {
                k *= 2;
                k = k / 10 + k % 10;
            }
            luhmSum += k;
        }
        return (luhmSum % 10 == 0) ? '0' : (char) ((10 - luhmSum % 10) + '0');
    }

    public static void main(String[] args) {
        System.out.println(checkBankCard("6228482372557704211"));
    }
}
```

