#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define STUDENT_FILE "students.txt"
#define CREDENTIAL_FILE "credentials.txt"

struct student {
    int roll;
    char name[50];
    float marks;
};

char currentRole[20];

/* ---------------- LOGIN SYSTEM ---------------- */
int loginSystem() {
    char user[50], pass[50], fuser[50], fpass[50], frole[20];
    
    printf("\n---- LOGIN ----\n");
    printf("Username: ");
    scanf("%s", user);
    printf("Password: ");
    scanf("%s", pass);

    FILE *fp = fopen(CREDENTIAL_FILE, "r");
    if (!fp) {
        printf("Error: credentials.txt not found!\n");
        return 0;
    }

    while (fscanf(fp, "%s %s %s", fuser, fpass, frole) != EOF) {
        if (strcmp(user, fuser) == 0 && strcmp(pass, fpass) == 0) {
            strcpy(currentRole, frole);
            fclose(fp);
            return 1;
        }
    }

    fclose(fp);
    return 0;
}

/* ---------------- ADD STUDENT (ADMIN ONLY) ---------------- */
void addStudent() {
    struct student s;
    FILE *fp = fopen(STUDENT_FILE, "a");

    printf("\nEnter Roll No: ");
    scanf("%d", &s.roll);
    printf("Enter Name: ");
    scanf("%s", s.name);
    printf("Enter Marks: ");
    scanf("%f", &s.marks);

    fprintf(fp, "%d %s %.2f\n", s.roll, s.name, s.marks);
    fclose(fp);

    printf("\nStudent Added Successfully!\n");
}

/* ---------------- DISPLAY STUDENTS ---------------- */
void displayStudents() {
    struct student s;
    FILE *fp = fopen(STUDENT_FILE, "r");

    printf("\n---- STUDENT LIST ----\n");

    while (fscanf(fp, "%d %s %f", &s.roll, s.name, &s.marks) != EOF) {
        printf("Roll: %d  Name: %s  Marks: %.2f\n", s.roll, s.name, s.marks);
    }

    fclose(fp);
}

/* ---------------- SEARCH STUDENT ---------------- */
void searchStudent() {
    int r;
    struct student s;
    int found = 0;

    FILE *fp = fopen(STUDENT_FILE, "r");

    printf("\nEnter Roll No to Search: ");
    scanf("%d", &r);

    while (fscanf(fp, "%d %s %f", &s.roll, s.name, &s.marks) != EOF) {
        if (s.roll == r) {
            printf("\nFOUND!\nRoll: %d  Name: %s  Marks: %.2f\n", s.roll, s.name, s.marks);
            found = 1;
        }
    }

    if (!found)
        printf("\nRecord Not Found!\n");

    fclose(fp);
}

/* ---------------- UPDATE STUDENT (ADMIN ONLY) ---------------- */
void updateStudent() {
    int r, found = 0;
    struct student s;

    FILE *fp = fopen(STUDENT_FILE, "r");
    FILE *temp = fopen("temp.txt", "w");

    printf("\nEnter Roll No to Update: ");
    scanf("%d", &r);

    while (fscanf(fp, "%d %s %f", &s.roll, s.name, &s.marks) != EOF) {
        if (s.roll == r) {
            printf("Enter New Name: ");
            scanf("%s", s.name);
            printf("Enter New Marks: ");
            scanf("%f", &s.marks);
            found = 1;
        }
        fprintf(temp, "%d %s %.2f\n", s.roll, s.name, s.marks);
    }

    fclose(fp);
    fclose(temp);

    remove(STUDENT_FILE);
    rename("temp.txt", STUDENT_FILE);

    if (found)
        printf("\nRecord Updated Successfully!\n");
    else
        printf("\nRecord Not Found!\n");
}

/* ---------------- DELETE STUDENT (ADMIN ONLY) ---------------- */
void deleteStudent() {
    int r, found = 0;
    struct student s;

    FILE *fp = fopen(STUDENT_FILE, "r");
    FILE *temp = fopen("temp.txt", "w");

    printf("\nEnter Roll No to Delete: ");
    scanf("%d", &r);

    while (fscanf(fp, "%d %s %f", &s.roll, s.name, &s.marks) != EOF) {
        if (s.roll == r) {
            found = 1;
        } else {
            fprintf(temp, "%d %s %.2f\n", s.roll, s.name, s.marks);
        }
    }

    fclose(fp);
    fclose(temp);

    remove(STUDENT_FILE);
    rename("temp.txt", STUDENT_FILE);

    if (found)
        printf("\nRecord Deleted Successfully!\n");
    else
        printf("\nRecord Not Found!\n");
}

/* ---------------- ADMIN MENU ---------------- */
void adminMenu() {
    int ch;

    do {
        printf("\n--- ADMIN MENU ---\n");
        printf("1. Add Student\n2. Display All\n3. Search\n4. Update\n5. Delete\n6. Logout\nEnter: ");
        scanf("%d", &ch);

        switch (ch) {
            case 1: addStudent(); break;
            case 2: displayStudents(); break;
            case 3: searchStudent(); break;
            case 4: updateStudent(); break;
            case 5: deleteStudent(); break;
        }
    } while (ch != 6);
}

/* ---------------- STAFF MENU ---------------- */
void staffMenu() {
    int ch;

    do {
        printf("\n--- STAFF MENU ---\n");
        printf("1. Display All\n2. Search\n3. Logout\nEnter: ");
        scanf("%d", &ch);

        switch (ch) {
            case 1: displayStudents(); break;
            case 2: searchStudent(); break;
        }
    } while (ch != 3);
}

/* ---------------- GUEST MENU ---------------- */
void guestMenu() {
    int ch;

    do {
        printf("\n--- GUEST MENU ---\n");
        printf("1. Display All\n2. Logout\nEnter: ");
        scanf("%d", &ch);

        switch (ch) {
            case 1: displayStudents(); break;
        }
    } while (ch != 2);
}

/* ---------------- MAIN FUNCTION ---------------- */
int main() {
    if (loginSystem()) {
        printf("\nLogin Successful! Role: %s\n", currentRole);

        if (strcmp(currentRole, "Admin") == 0)
            adminMenu();
        else if (strcmp(currentRole, "Staff") == 0)
            staffMenu();
        else
            guestMenu();
    } else {
        printf("\nAccess Denied! Invalid Username or Password.\n");
    }

    return 0;
}
