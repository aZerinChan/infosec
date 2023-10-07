# DES


## 代码

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <openssl/des.h>

void print_hex(const char *tag, const unsigned char *data, int len) {
    printf("%s:", tag);
    for (int i = 0; i < len; ++i) {
        printf(" %02x", data[i]);
    }
    printf("\n");
}

void print_binary(const char *tag, const unsigned char *data, int len) {
    printf("%s:", tag);
    for (int i = 0; i < len; ++i) {
        for (int j = 7; j >= 0; --j) {
            printf("%d", (data[i] >> j) & 1);
        }
        printf(" ");
    }
    printf("\n");
}

int main() {
    // 固定密钥，改变明文
    unsigned char key[8] = {0x01, 0x23, 0x45, 0x67, 0x89, 0xab, 0xcd, 0xef};
    unsigned char plaintext[8] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};
    unsigned char ciphertext1[8], ciphertext2[8];
    DES_key_schedule ks;
    DES_set_key_unchecked(&key, &ks);
    DES_ecb_encrypt(&plaintext, &ciphertext1, &ks, DES_ENCRYPT);
    for (int i = 0; i < 64; ++i) {
        memcpy(&plaintext, &ciphertext1, sizeof(plaintext));
        plaintext[i / 8] ^= 1 << (i % 8);
        DES_ecb_encrypt(&plaintext, &ciphertext2, &ks, DES_ENCRYPT);
        int diff = 0;
        for (int j = 0; j < 8; ++j) {
            if (ciphertext1[j] != ciphertext2[j]) {
                diff += 8 - __builtin_popcount(ciphertext1[j] ^ ciphertext2[j]);
            }
        }
        printf("改变%d位，输出密文位数改变：%d\n", i + 1, diff);
    }

    // 固定明文，改变密钥
    memset(&plaintext, 0, sizeof(plaintext));
    unsigned char key1[8], key2[8];
    memcpy(&key1, &key, sizeof(key));
    DES_set_key_unchecked(&key1, &ks);
    DES_ecb_encrypt(&plaintext, &ciphertext1, &ks, DES_ENCRYPT);
    for (int i = 0; i < 64; ++i) {
        memcpy(&key2, &key1, sizeof(key));
        key2[i / 8] ^= 1 << (i % 8);
        DES_set_key_unchecked(&key2, &ks);
        DES_ecb_encrypt(&plaintext, &ciphertext2, &ks, DES_ENCRYPT);
        int diff = 0;
        for (int j = 0; j < 8; ++j) {
            if (ciphertext1[j] != ciphertext2[j]) {
                diff += 8 - __builtin_popcount(ciphertext1[j] ^ ciphertext2[j]);
            }
        }
        printf("改变%d位，输出密文位数改变：%d\n", i + 1, diff);
    return 0;
}

```

## 说明

代码中首先定义了两个辅助函数 print_hex 和 print_binary，分别用于以十六进制和二进制格式打印数据。然后在 main 函数中，定义了固定密钥、改变明文和固定明文、改变密钥的两个场景，分别进行测试。

在第一个场景中，使用 DES_key_schedule 结构体初始化密钥，并使用 DES_ecb_encrypt 函数对固定的明文进行加密，得到输出的密文 ciphertext1。接着，通过循环改变输入的明文的每一位，得到新的密文 ciphertext2，并计算两个密文之间的差异位数，输出结果。

在第二个场景中，先使用相同的明文和固定的密钥 key 进行加密，得到输出的密文 ciphertext1。接着，通过循环改变密钥的每一位，得到新的密钥 key2，并使用 DES_set_key_unchecked 函数重新设置密钥，再使用 DES_ecb_encrypt 函数对相同的明文进行加密，得到新的密文 ciphertext2，并计算两个密文之间的差异位数，输出结果。