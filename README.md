# SPC-AAT
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

/* Structures */
struct Player {
    char name[30];
    int health;
    int ammo;
    int score;
    int level;
};

struct Enemy {
    int health;
    int alive;
};

/* Function Declarations */
void showStatus(struct Player *p);
void shootEnemy(struct Player *p, struct Enemy enemies[], int count);
void enemyAttack(struct Player *p, struct Enemy enemies[], int count);
void reloadGun(struct Player *p);
void healPlayer(struct Player *p);
int enemiesAlive(struct Enemy enemies[], int count);
void saveGame(struct Player *p, char result[]);

int main() {
    struct Player player;
    struct Enemy enemies[5];
    int choice, i, enemyCount;

    srand(time(NULL));

    printf("Enter Player Name: ");
    scanf("%s", player.name);

    player.health = 100;
    player.ammo = 10;
    player.score = 0;

    /* Level Selection */
    printf("\nSelect Level (1-Easy | 2-Medium | 3-Hard): ");
    scanf("%d", &player.level);

    if (player.level == 1)
        enemyCount = 3;
    else if (player.level == 2)
        enemyCount = 4;
    else
        enemyCount = 5;

    /* Initialize Enemies */
    for (i = 0; i < enemyCount; i++) {
        enemies[i].health = 40 + (player.level * 10);
        enemies[i].alive = 1;
    }

    /* Game Loop */
    while (player.health > 0 && enemiesAlive(enemies, enemyCount) > 0) {
        printf("\n===== FREE FIRE LEVEL %d =====\n", player.level);
        printf("1. Shoot Enemy\n");
        printf("2. Reload Gun\n");
        printf("3. Heal Player\n");
        printf("4. Show Status\n");
        printf("5. Exit Game\n");
        printf("Enter choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                shootEnemy(&player, enemies, enemyCount);
                enemyAttack(&player, enemies, enemyCount);
                break;
            case 2:
                reloadGun(&player);
                break;
            case 3:
                healPlayer(&player);
                break;
            case 4:
                showStatus(&player);
                break;
            case 5:
                saveGame(&player, "Quit");
                exit(0);
            default:
                printf("Invalid Choice!\n");
        }
    }

    /* Result */
    if (player.health <= 0) {
        printf("\n💀 GAME OVER!\n");
        saveGame(&player, "Lost");
    } else {
        printf("\n🏆 LEVEL CLEARED!\n");
        player.score += 50;
        saveGame(&player, "Won");
    }

    return 0;
}

/* Functions */

void showStatus(struct Player *p) {
    printf("\n--- Player Status ---\n");
    printf("Name   : %s\n", p->name);
    printf("Health : %d\n", p->health);
    printf("Ammo   : %d\n", p->ammo);
    printf("Score  : %d\n", p->score);
}

void shootEnemy(struct Player *p, struct Enemy enemies[], int count) {
    int i;

    if (p->ammo <= 0) {
        printf("❌ No ammo! Reload first.\n");
        return;
    }

    for (i = 0; i < count; i++) {
        if (enemies[i].alive) {
            int damage = rand() % 20 + 10;
            enemies[i].health -= damage;
            p->ammo--;
            printf("🔫 Shot enemy %d for %d damage\n", i + 1, damage);

            if (enemies[i].health <= 0) {
                enemies[i].alive = 0;
                p->score += 10;
                printf("☠ Enemy %d defeated!\n", i + 1);
            }
            break;
        }
    }
}

void enemyAttack(struct Player *p, struct Enemy enemies[], int count) {
    int i;
    for (i = 0; i < count; i++) {
        if (enemies[i].alive) {
            int damage = rand() % 10 + (5 * p->level);
            p->health -= damage;
            printf("⚠ Enemy attacked you for %d damage!\n", damage);
            break;
        }
    }
}

void reloadGun(struct Player *p) {
    p->ammo = 10;
    printf("🔄 Gun Reloaded\n");
}

void healPlayer(struct Player *p) {
    if (p->health < 100) {
        p->health += 20;
        if (p->health > 100)
            p->health = 100;
        printf("💉 Health Restored\n");
    } else {
        printf("❤️ Health Already Full\n");
    }
}

int enemiesAlive(struct Enemy enemies[], int count) {
    int i, alive = 0;
    for (i = 0; i < count; i++) {
        if (enemies[i].alive)
            alive++;
    }
    return alive;
}

/* File Handling */
void saveGame(struct Player *p, char result[]) {
    FILE *fp;
    fp = fopen("game_record.txt", "a");

    if (fp == NULL) {
        printf("File Error!\n");
        return;
    }

    fprintf(fp,
        "Player: %s | Level: %d | Score: %d | Result: %s\n",
        p->name, p->level, p->score, result);

    fclose(fp);
}
