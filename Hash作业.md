# Hash作业

## 代码
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <Windows.h>

#define MAX_PATH_LENGTH 256
#define HASH_TABLE_SIZE 10007

typedef struct {
    char path[MAX_PATH_LENGTH];
    DWORD hash;
    struct Node* next;
} Node;

Node* hashTable[HASH_TABLE_SIZE];

DWORD calculateHash(const char* filePath) {
    HANDLE fileHandle = CreateFile(filePath, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
    if (fileHandle == INVALID_HANDLE_VALUE) {
        printf("Error opening file: %s\n", filePath);
        return 0;
    }

    DWORD hash = 0;
    BYTE buffer[4096];
    DWORD bytesRead;

    while (ReadFile(fileHandle, buffer, sizeof(buffer), &bytesRead, NULL)) {
        if (bytesRead == 0)
            break;
        
        for (DWORD i = 0; i < bytesRead; i++) {
            hash = ((hash << 5) + hash) ^ buffer[i];
        }
    }

    CloseHandle(fileHandle);

    return hash;
}

void insertNode(const char* filePath, DWORD hash) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    strcpy(newNode->path, filePath);
    newNode->hash = hash;
    newNode->next = NULL;

    int index = hash % HASH_TABLE_SIZE;

    if (hashTable[index] == NULL) {
        hashTable[index] = newNode;
    } else {
        Node* currentNode = hashTable[index];
        while (currentNode->next != NULL) {
            currentNode = currentNode->next;
        }
        currentNode->next = newNode;
    }
}

void findDuplicateFiles() {
    for (int i = 0; i < HASH_TABLE_SIZE; i++) {
        if (hashTable[i] != NULL) {
            Node* currentNode = hashTable[i];
            while (currentNode != NULL) {
                Node* nextNode = currentNode->next;
                Node* duplicateNode = currentNode->next;

                while (duplicateNode != NULL) {
                    if (duplicateNode->hash == currentNode->hash) {
                        // Compare the contents of the files
                        if (strcmp(duplicateNode->path, currentNode->path) != 0) {
                            printf("Duplicate files found:\n");
                            printf("%s\n", duplicateNode->path);
                            printf("%s\n", currentNode->path);
                            printf("\n");
                        }
                    }

                    duplicateNode = duplicateNode->next;
                }

                currentNode = nextNode;
            }
        }
    }
}

int main() {
    // Initialize the hash table
    for (int i = 0; i < HASH_TABLE_SIZE; i++) {
        hashTable[i] = NULL;
    }

    // Directory to scan for duplicate files
    const char* directory = "C:\\Path\\To\\Directory";

    WIN32_FIND_DATA findData;
    HANDLE findHandle = FindFirstFile(directory, &findData);
    if (findHandle == INVALID_HANDLE_VALUE) {
        printf("Error opening directory: %s\n", directory);
        return 1;
    }

    do {
        if (findData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY) {
            // Skip directories
            continue;
        }

        char filePath[MAX_PATH_LENGTH];
        sprintf(filePath, "%s\\%s", directory, findData.cFileName);

        DWORD hash = calculateHash(filePath);
        if (hash != 0) {
            insertNode(filePath, hash);
        }
    } while (FindNextFile(findHandle, &findData));

    FindClose(findHandle);

    findDuplicateFiles();

    return 0;
}
```

## 说明

在上述代码中，首先定义了一个Node结构体，用于表示哈希表中的节点。节点包含文件路径、哈希值和指向下一个节点的指针。然后定义了一个哈希表hashTable，使用拉链法解决哈希冲突。

calculateHash函数用于计算文件的哈希值。使用了Windows API中的CreateFile和ReadFile函数来读取文件内容，并对内容进行哈希计算。这里使用了简单的哈希算法，每次读取一个字节进行哈希运算，最终得到文件的哈希值。

insertNode函数用于将文件路径和哈希值插入到哈希表中。根据哈希值计算得到插入位置，如果该位置为空，则直接插入节点；否则，遍历链表直到找到合适的插入位置。

findDuplicateFiles函数用于遍历哈希表，查找具有相同哈希值的文件。对于每个哈希值，遍历对应链表的节点，并与后续节点进行比较。如果发现哈希值相同且文件内容不同的节点，说明找到了重复文件。

在main函数中，首先初始化哈希表，然后使用FindFirstFile和FindNextFile函数遍历指定目录下的所有文件。对于每个文件，我们调用calculateHash函数计算哈希值，并使用insertNode函数将路径和哈希值插入到哈希表中。

最后，调用findDuplicateFiles函数查找重复文件并输出结果。