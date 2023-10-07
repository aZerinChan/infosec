# RSA

## 代码
```
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>

int gcd(int a, int b) {
    if (b == 0) {
        return a;
    }
    return gcd(b, a % b);
}

int mod_pow(int base, int exponent, int modulus) {
    int result = 1;
    while (exponent > 0) {
        if (exponent % 2 == 1) {
            result = (result * base) % modulus;
        }
        base = (base * base) % modulus;
        exponent = exponent / 2;
    }
    return result;
}

int main() {
    int p, q, n, phi, e, d, message, cipher, decrypted;

    srand(time(0)); // 设置随机种子

    // 生成两个随机素数p和q
    p = rand() % 100 + 1; // 在1~100之间随机生成p
    while (1) {
        int is_prime = 1;
        for (int i = 2; i <= sqrt(p); i++) {
            if (p % i == 0) {
                is_prime = 0;
                break;
            }
        }
        if (is_prime) {
            break;
        }
        p++;
    }
    q = rand() % 100 + 1; // 在1~100之间随机生成q
    while (1) {
        int is_prime = 1;
        for (int i = 2; i <= sqrt(q); i++) {
            if (q % i == 0) {
                is_prime = 0;
                break;
            }
        }
        if (is_prime) {
            break;
        }
        q++;
    }

    // 计算n和phi(n)
    n = p * q;
    phi = (p - 1) * (q - 1);

    // 选择公钥e
    e = 2;
    while (e < phi) {
        if (gcd(e, phi) == 1) {
            break;
        }
        e++;
    }

    // 计算私钥d
    int k = 1;
    while (1) {
        if ((k * phi + 1) % e == 0) {
            d = (k * phi + 1) / e;
            break;
        }
        k++;
    }

    // 加密
    printf("请输入要加密的消息（小于 %d）：", n);
    scanf("%d", &message);
    cipher = mod_pow(message, e, n);
    printf("加密后的密文为：%d\n", cipher);

    // 解密
    decrypted = mod_pow(cipher, d, n);
    printf("解密后的消息为：%d\n", decrypted);

    return 0;
}
```
## 说明

在这个实现中，我们使用了以下函数：

- gcd()：计算两个数的最大公约数。
- mod_pow()：计算一个数的幂的模数。

在主函数中，我们首先生成了两个随机素数p和q，并计算了n和phi(n)。然后选择公钥e，并计算了私钥d。接下来，我们通过输入一个消息，使用公钥加密该消息，并使用私钥解密该消息。

在加密过程中，我们需要选择一个小于n的整数作为消息，然后计算其e次幂再对n取模。这个结果就是加密后的密文。

在解密过程中，我们需要使用私钥d来计算密文的d次幂再对n取模。这个结果就是解密后的消息。

需要注意的是，在实际使用中，RSA算法需要使用非常大的素数和整数，以确保足够的安全性。这里我们仅使用了较小的素数和整数作为示例，实际使用中需要使用更大的数值。同时，RSA算法的实现还需要考虑一些其他因素，如填充方案等，以确保安全性和实用性。





