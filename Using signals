#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <time.h>
#include <sys/wait.h>

int secret_number;
int attempts = 0;
pid_t child_pid;

void guess_handler(int sig) {
    if (sig == SIGUSR1) {
        printf("Угадал число %d за %d попыток!\n", secret_number, attempts);
        exit(0); // Завершить второй процесс
    } else if (sig == SIGUSR2) {
        printf("Не угадал, пробую снова...\n");
    }
}

void play_game(int n) {
    srand(time(NULL));
    secret_number = rand() % n + 1; // Загадать число от 1 до N
    printf("Первый игрок загадал число от 1 до %d.\n", n);
    
    while (1) {
        attempts++;
        int guess = rand() % n + 1; // Второй игрок угадывает
        printf("Второй игрок пытается угадать: %d\n", guess);
        
        if (guess == secret_number) {
            kill(child_pid, SIGUSR1); // Угадал
            return; // Завершить игру
        } else {
            kill(child_pid, SIGUSR2); // Не угадал
        }
        
        sleep(1); // Небольшая пауза для наглядности
    }
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Использование: %s <N>\n", argv[0]);
        return 1;
    }

    int n = atoi(argv[1]);
    
    signal(SIGUSR1, guess_handler);
    signal(SIGUSR2, guess_handler);

    for (int i = 0; i < 10; i++) { // Играем 10 раз
        child_pid = fork();
        
        if (child_pid < 0) {
            perror("Ошибка fork");
            return 1;
        }

        if (child_pid == 0) { // Второй процесс
            while (1) {
                pause(); // Ожидание сигнала
            }
        } else { // Первый процесс
            play_game(n);
            wait(NULL); // Ждем завершения второго процесса
            printf("Игра окончена. Начинаем заново...\n");
            attempts = 0; // Сбросить счетчик попыток
            kill(child_pid, SIGKILL); // Завершить второго игрока
        }
    }

    return 0;
}
