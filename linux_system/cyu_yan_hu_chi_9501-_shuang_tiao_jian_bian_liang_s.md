# C語言互斥鎖-雙條件變量實現循環打印


```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <error.h>
#include <unistd.h>
#include <pthread.h>

int number[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
pthread_mutex_t mutex;
pthread_cond_t cond;
pthread_cond_t cond2;
int i = 0;      //數組下標
void* read_even(void* ptr)        //此線程用來打印偶數
{
    do {
        pthread_mutex_lock(&mutex);//鎖定互斥鎖

        if (i % 2 == 0) { //對應是奇數，就阻塞線程
            /*阻塞線程,等待另外一個線程發送信號，同時為公共數據區解鎖*/
            pthread_cond_wait(&cond, &mutex);
        } else if (i % 2 == 1) { //偶數，打印
            printf("thread2--->number[%d]=%d\n", i, number[i]);
            i++;
            pthread_cond_signal(&cond2);//條件改變，喚醒阻塞的線程
        }

        pthread_mutex_unlock(&mutex);//打開互斥鎖
    } while (i <= 9);

    pthread_exit(NULL);
}

void* read_odd(void* ptr)     //此線程用來打印奇數
{
    do {
        pthread_mutex_lock(&mutex);

        if (i % 2 == 1) {
            pthread_cond_wait(&cond2, &mutex);
        } else if (i % 2 == 0) {
            printf("thread1--->number[%d]=%d\n", i, number[i]);
            i++;
            pthread_cond_signal(&cond);//條件改變，喚醒阻塞的線程
        }

        pthread_mutex_unlock(&mutex);
    } while (i <= 9);

    pthread_exit(NULL);
}
int main(int argc, char** argv)
{
    pthread_t id, id2;
    int ret = -1;
    pthread_cond_init(&cond, NULL); //初始化條件變量
    pthread_cond_init(&cond2, NULL);
    pthread_mutex_init(&mutex, NULL); //初始化互斥鎖
    ret = pthread_create(&id, NULL, read_odd, NULL);

    if (ret != 0) {
        printf("Create pthread error!\n");
        exit(1);
    }

    ret = pthread_create(&id2, NULL, read_even,
                         NULL); //返回值為0，創建成功

    if (ret != 0) {
        printf("Create pthread error!\n");
        exit(1);
    }

    pthread_join(id, NULL);
    pthread_join(id2, NULL);
    pthread_mutex_destroy(&mutex);//銷燬互斥鎖
    pthread_cond_destroy(&cond);//銷燬條件變量
    pthread_cond_destroy(&cond2);
    return 0;
}

```

