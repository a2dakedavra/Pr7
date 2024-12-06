#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <time.h>
#include <sys.wait.h>

#define BUFFER_SIZE 32

void play_game(int n, int write_fd, int read_fd) {
    srand(time(NULL));
    int secret_number = rand() % n + 1; // Загадать число от 1 до N
    printf("Первый игрок загадал число от 1 до %d.\n", n);
    
    char buffer[BUFFER_SIZE];
    int attempts = 0;

    while (1) {
        attempts++;
        
        int guess = rand() % n + 1; // Второй игрок угадывает
        printf("Второй игрок пытается угадать: %d\n", guess);
        
        write(write_fd, &guess, sizeof(guess)); // Отправить предположение
        
        read(read_fd, buffer, BUFFER_SIZE); // Ожидать ответ
        
        if (strcmp(buffer, "win") == 0) {
            printf("Угадал число %d за %d попыток!\n", secret_number, attempts);
            return; // Завершить игру
        } else {
            printf("Не угадал, пробую снова...\n");
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
    
    for (int i = 0; i < 10; i++) { // Играем 10 раз
        int pipe_fd[2];
        
        if (pipe(pipe_fd) == -1) {
            perror("Ошибка создания канала");
            return 1;
        }

        pid_t child_pid = fork();
        
        if (child_pid < 0) {
            perror("Ошибка fork");
            return 1;
        }

        if (child_pid == 0) { // Второй процесс
            close(pipe_fd[1]); // Закрыть запись в канале
            
            char buffer[BUFFER_SIZE];
            
            while (read(pipe_fd[0], buffer, sizeof(int)) > 0) { 
                int guess;
                memcpy(&guess, buffer, sizeof(int));
                
                if (guess == -1) break; // Завершение по сигналу
                
                printf("Второй игрок получил предположение: %d\n", guess);
                
                if (guess == secret_number) {
                    strcpy(buffer, "win");
                    write(pipe_fd[0], buffer, BUFFER_SIZE); 
                    exit(0);
                } else {
                    strcpy(buffer, "continue");
                    write(pipe_fd[0], buffer, BUFFER_SIZE);
                }
                
                sleep(1); // Небольшая пауза для наглядности
            }
            
            close(pipe_fd[0]); 
            exit(0);
            
        } else { // Первый процесс
            close(pipe_fd[0]); // Закрыть чтение из канала
            
            play_game(n, pipe_fd[1], pipe_fd[0]);
            
            printf("Игра окончена. Начинаем заново...\n");
            
            kill(child_pid, SIGKILL); // Завершить второго игрока
        }
    }

    return 0;
}
