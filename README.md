#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FILE_NAME "passwords.dat"
#define KEY 0xAA  // XOR encryption key

void encryptDecrypt(char *str) {
    while (*str) {
        *str ^= KEY;
        str++;
    }
}

void addPassword() {
    FILE *file = fopen(FILE_NAME, "a");
    if (!file) {
        printf("Error opening file!\n");
        return;
    }

    char site[50], username[50], password[50];
    printf("Enter site: ");
    scanf("%s", site);
    printf("Enter username: ");
    scanf("%s", username);
    printf("Enter password: ");
    scanf("%s", password);

    encryptDecrypt(password);
    fprintf(file, "%s %s %s\n", site, username, password);
    fclose(file);
    printf("Password saved successfully!\n");
}

void viewPasswords() {
    FILE *file = fopen(FILE_NAME, "r");
    if (!file) {
        printf("No passwords found.\n");
        return;
    }
    char site[50], username[50], password[50];
    printf("Stored passwords:\n");
    while (fscanf(file, "%s %s %s", site, username, password) != EOF) {
        encryptDecrypt(password);
        printf("Site: %s, Username: %s, Password: %s\n", site, username, password);
    }
    fclose(file);
}

void updatePassword() {
    FILE *file = fopen(FILE_NAME, "r");
    if (!file) {
        printf("No passwords found.\n");
        return;
    }

    FILE *tempFile = fopen("temp.dat", "w");
    char site[50], username[50], password[50];
    char searchSite[50], newPassword[50];
    int found = 0;
    
    printf("Enter site to update: ");
    scanf("%s", searchSite);
    
    while (fscanf(file, "%s %s %s", site, username, password) != EOF) {
        encryptDecrypt(password);
        if (strcmp(site, searchSite) == 0) {
            found = 1;
            printf("Enter new password: ");
            scanf("%s", newPassword);
            encryptDecrypt(newPassword);
            fprintf(tempFile, "%s %s %s\n", site, username, newPassword);
        } else {
            encryptDecrypt(password);
            fprintf(tempFile, "%s %s %s\n", site, username, password);
        }
    }
    fclose(file);
    fclose(tempFile);
    remove(FILE_NAME);
    rename("temp.dat", FILE_NAME);
    
    if (found)
        printf("Password updated successfully!\n");
    else
        printf("Site not found!\n");
}

void deletePassword() {
    FILE *file = fopen(FILE_NAME, "r");
    if (!file) {
        printf("No passwords found.\n");
        return;
    }

    FILE *tempFile = fopen("temp.dat", "w");
    char site[50], username[50], password[50];
    char searchSite[50];
    int found = 0;
    
    printf("Enter site to delete: ");
    scanf("%s", searchSite);
    
    while (fscanf(file, "%s %s %s", site, username, password) != EOF) {
        encryptDecrypt(password);
        if (strcmp(site, searchSite) != 0) {
            encryptDecrypt(password);
            fprintf(tempFile, "%s %s %s\n", site, username, password);
        } else {
            found = 1;
        }
    }
    fclose(file);
    fclose(tempFile);
    remove(FILE_NAME);
    rename("temp.dat", FILE_NAME);
    
    if (found)
        printf("Password deleted successfully!\n");
    else
        printf("Site not found!\n");
}

int main() {
    int choice;
    do {
        printf("\nPassword Manager\n");
        printf("1. Add Password\n");
        printf("2. View Passwords\n");
        printf("3. Update Password\n");
        printf("4. Delete Password\n");
        printf("5. Exit\n");
        printf("Enter choice: ");
        scanf("%d", &choice);
        
        switch (choice) {
            case 1:
                addPassword();
                break;
            case 2:
                viewPasswords();
                break;
            case 3:
                updatePassword();
                break;
            case 4:
                deletePassword();
                break;
            case 5:
                printf("Exiting...\n");
                break;
            default:
                printf("Invalid choice!\n");
        }
    } while (choice != 5);
    return 0;
}
