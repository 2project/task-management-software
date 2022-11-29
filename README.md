# task-management-software
Task management software
#include<stdio.h>
#include<stdlib.h>
#include<stdbool.h>
#include<string.h>
#define VECTOR_GROW_FACTOR 10
#define CSV_DELIM ","
#define USERS_FILE "Users.csv"
#define TASKS_FILE "Tasks.csv"
#define TASK_ASSIGNMENT_FILE "Assignment.csv"
/*C doesn't have an array that can grow dynamically.below is a simple implementation of dynamic array*/
typedef struct {
	size_t size;      // total allocated size.
	size_t used_size; // size that is used. if used size is almost equal to allocated size , then we increase the size by reallocating the array.
	void** elements; // we can store elemetnt of any type, hence void ** representing pointer of pointers.
} vector;
int vector_add(vector* vec, void* item)
{
	if(vec->used_size + 1 >= vec->size)  // if used_size is close to allocated size.
	{
		vec->size += VECTOR_GROW_FACTOR; // resize the array by VECTOR_GROW_SIZE.
		vec->elements = realloc(vec->elements, vec->size * sizeof(item));
	}
	vec->elements[vec->used_size] = item;
	vec->used_size++;
}
// reading lines from file and stdin sometimes add trailing newline chars. this helps to remove unnecessary newline.
char *remove_trailing_newline(char *str)
{
	char *startPtr = str;
	int len = strlen(str);
	if(len != 0)
	{
		if (str[len-1] == '\n'){
			str[len-1] = '\0';
		}
	}
	return startPtr;
}
typedef struct{
	int user_id;
	char* user_name;
	char* user_designation;
	bool is_deleted; // deleted records are not persisted to file.
}User;
typedef struct{
	int task_id;
	char* task_name;
	char* task_description;
	char* task_deadline;
	bool is_deleted; // deleted records are not persisted to file.
}Task;
typedef struct{
	int user_id;
	int task_id;
	bool is_deleted; // deleted records are not persisted to file.
}TaskAssignment;
vector *users_list;
vector *tasks_list;
vector *task_assignment_list;
/*reads data from file and adds it to users_list array.*/
int load_users(){
	FILE *fp;
	char *line;
	char *token;
	size_t len=0;
	int read;
	int lineno=0;
	User *user;
	users_list = malloc(sizeof(vector));
	users_list->used_size = 0;
	users_list->size = 0;
	fp = fopen(USERS_FILE, "r");
	if(fp == NULL)
	{
		printf("error failed to read users data from file. no data is loaded from file.");
		return 1;
	}
	while ((read = getline(&line, &len, fp)) != -1){
		char *lineCopy = strdup(line);
		lineno++;
		user = malloc(sizeof(User));
		token = strtok(lineCopy, CSV_DELIM);
		if (token == NULL)
		{
			continue;
		}
		user->user_id = atoi(token);
		token = strtok(NULL, CSV_DELIM);
		if (token == NULL){
			continue;
		}
		user->user_name= malloc(sizeof(strlen(token)) + 2);
		strcpy(user->user_name, token);
		remove_trailing_newline(user->user_name);
		token = strtok(NULL, CSV_DELIM);
		if(token == NULL)
		{
			continue;
		}
		user->user_designation = malloc(sizeof(strlen(token)) + 2);
		strcpy(user->user_designation, token);
		remove_trailing_newline(user->user_designation);
		
		user->is_deleted = 0;
		vector_add(users_list, user);
	}
	printf("info completed reading users data from file");
}
User *find_user(int user_id){
	if (users_list == NULL)
	{
		return NULL;
	}
	for(int i=0;i<users_list->used_size;i++)
	{
		User *currentUser = (User *)users_list->elements[i];
		if (currentUser->user_id == user_id && currentUser->is_deleted == 0)
		{
			return currentUser;
		}
	}
	return NULL;
}
int add_user(int user_id, char *user_name, char *user_designation)
{
	if (find_user(user_id) != NULL)
	{
		printf("error user already exists. cannot add new user with duplicate id");
		return 1;
	}
	User *user = malloc(sizeof(User));
	user->user_id = user_id;
	user->user_name = malloc(sizeof(strlen(user_name)));
	strcpy(user->user_name, user_name);
	user->user_designation = malloc(sizeof(strlen(user_designation)));
	strcpy(user->user_designation, user_designation);
	vector_add(users_list, user);
}
int update_user(int user_id, char *user_name, char *user_designation)
{
	User *user = find_user(user_id);
	if ( user == NULL)
	{
		printf("error user not found to update");
		return 1;
	}
	free(user->user_name);
	free(user->user_designation);
	user->user_name = malloc(sizeof(strlen(user_name)));
	strcpy(user->user_name, user_name);
	user->user_designation = malloc(sizeof(strlen(user_designation)));
	strcpy(user->user_designation, user_designation);
}
int delete_user(int user_id)
{
	User *user = find_user(user_id);
	if (user == NULL)
	{
		printf("info user does not exist to delete.");
		return 1;
	}
	user->is_deleted = 1;
	return 0;
}
/* save users data to file */
int commit_users_to_file(){
	FILE *fp;
	if (users_list == NULL)
	{
		printf("error No memory allocaed to users_list. failed to write users to file");
		return 1;
	}
	fp = fopen(USERS_FILE, "w+");
	for(int i=0;i<users_list->used_size; i++)
	{
		User *user = (User *)users_list->elements[i];
		fprintf(fp, "%d,%s,%s\n", user->user_id, user->user_name, user->user_designation);
	}
	fclose(fp);
	return 0;
}
/* read tasks from file */
int load_tasks(){
	FILE *fp;
	char *line;
	char *token;
	size_t len=0;
	int read;
	int lineno=0;
	Task *task;
	tasks_list = malloc(sizeof(vector));
	tasks_list->used_size = 0;
	tasks_list->size = 0;
	fp = fopen(TASKS_FILE, "r");
	if(fp == NULL)
	{
		printf("error failed to read tasks data from file. no data is loaded from file.");
		return 1;
	}
	while ((read = getline(&line, &len, fp)) != -1){
		char *lineCopy  =strdup(line);
		lineno++;
		task = malloc(sizeof(Task));
		token = strtok(lineCopy, CSV_DELIM);
		if (token == NULL)
		{
			continue;
		}
		task->task_id = atoi(token);
		token = strtok(NULL, CSV_DELIM);
		if (token == NULL){
			continue;
		}
		task->task_name= malloc(sizeof(strlen(token)));
		strcpy(task->task_name, token);
		remove_trailing_newline(task->task_name);
		token = strtok(NULL, CSV_DELIM);
		if(token == NULL)
		{
			continue;
		}
		task->task_description = malloc(sizeof(strlen(token)));
		strcpy(task->task_description, token);
		remove_trailing_newline(task->task_description);
		token = strtok(NULL, CSV_DELIM);
		if(token == NULL)
		{
			continue;
		}
		task->task_deadline = malloc(sizeof(strlen(token)));
		strcpy(task->task_deadline, token);
		remove_trailing_newline(task->task_deadline);
		task->is_deleted = 0;
		vector_add(tasks_list, task);
	}
}
/* find the task with given task Id. is_deleted=1 will be ignored.*/
Task *find_task(int task_id)
{
	if (tasks_list == NULL)
	{
		return NULL;
	}
	for(int i=0;i<tasks_list->used_size;i++)
	{
		Task *task = (Task *)tasks_list->elements[i];
		if (task->task_id == task_id && task->is_deleted == 0)
		{
			return task;
		}
	}
	return NULL;
}
/* add new task to list. */
int add_task(int task_id, char *task_name, char *task_description, char *task_deadline)
{
	if (find_task(task_id) != NULL)
	{
		printf("error cannot add task. task alrady exists");
		return 1;
	}
	Task *task = malloc(sizeof(Task));
	task->task_id = task_id;
	task->task_name = malloc(sizeof(strlen(task_name)));
	strcpy(task->task_name, task_name);
	task->task_description = malloc(sizeof(strlen(task_description)));
	strcpy(task->task_description, task_description);
	task->task_deadline = malloc(sizeof(strlen(task_deadline)));
	strcpy(task->task_deadline, task_deadline);
	task->is_deleted = 0;
	vector_add(tasks_list, task);
}
/* update task. */
int update_task(int task_id, char *task_name, char *task_description, char *task_deadline)
{
	Task *task = find_task(task_id);
	if (task == NULL)
	{
		printf("error cannot update task. task doesn't exists");
		return 1;
	}
	free(task->task_name);
	free(task->task_description);
	free(task->task_deadline);
	task->task_name = malloc(sizeof(strlen(task_name)));
	strcpy(task->task_name, task_name);
	task->task_description = malloc(sizeof(strlen(task_description)));
	strcpy(task->task_description, task_description);
	task->task_deadline = malloc(sizeof(strlen(task_deadline)));
	strcpy(task->task_deadline, task_deadline);
	return 0;
}
/* delete tasks. we are only doing soft delete here. records with is_deleted = 1 will be ignored.
   soft delete is used as it is bit expensive to delete any element in an array.*/
int delete_task(int task_id)
{
	Task *task = find_task(task_id);
	if (task == NULL)
	{
		printf("error No task found to delete.");
		return 1;
	}
	task->is_deleted = 1;
}
/* save tasks data to file. */
int commit_tasks_to_file(){
	FILE *fp;
	if (tasks_list == NULL)
	{
		printf("error no memory allocated to tasks to write. failed to write task to file");
	}
	fp = fopen(TASKS_FILE, "w+");
	for(int i=0;i<tasks_list->used_size; i++)
	{
		Task *task = (Task *)tasks_list->elements[i];
		fprintf(fp, "%d,%s,%s,%s\n", task->task_id, task->task_name, task->task_description, task->task_deadline);
	}
	fclose(fp);
}
/* load task assignemnts from csv file */
int load_task_assignment(){
	FILE *fp;
	char *line;
	char *token;
	size_t len=0;
	int read;
	int lineno=0;
	TaskAssignment *ta;
	task_assignment_list = malloc(sizeof(vector));
	task_assignment_list->used_size = 0;
	task_assignment_list->size = 0;
	fp = fopen(TASK_ASSIGNMENT_FILE, "r");
	if(fp == NULL)
	{
		printf("error failed to read task assignment data. no data is loaded from file.");
		return 1;
	}
	while ((read = getline(&line, &len, fp)) != -1){
		char *lineCopy = strdup(line);
		lineno++;
		ta = malloc(sizeof(TaskAssignment));
		
		token = strtok(lineCopy, CSV_DELIM);
		if (token == NULL)
		{
			continue;
		}
		ta->task_id = atoi(token);
		token = strtok(NULL, CSV_DELIM);
		if (token == NULL){
			continue;
		}
		ta->user_id = atoi(token);
		vector_add(task_assignment_list, ta);
	}
}
/* add new task assignment to list */
int add_task_assignment(int task_id, int user_id)
{
	TaskAssignment *ta = malloc(sizeof(TaskAssignment));
	if (task_assignment_list == NULL)
	{
		printf("null list \n");
	}
	printf("in add ta %d %d", task_id, user_id);
	// printf("in add ta %d %d", task_id, user_id);
	ta->user_id = user_id;
	ta->task_id = task_id;
	vector_add(task_assignment_list, ta);
	return 0;
}
/*saves task_assigment to file.reads each task assignment from task_assignment_list and write to file 
using fprintf in comma seperated format.*/
int commit_task_assignment_to_file(){
	FILE *fp;
	if (task_assignment_list == NULL)
	{
		printf("error no memory allocated to task assignments. failed to write task assignment to file.");
	}
	fp = fopen(TASK_ASSIGNMENT_FILE, "w+");
	for(int i=0;i<task_assignment_list->used_size; i++)
	{
		TaskAssignment *ta = (TaskAssignment *)task_assignment_list->elements[i];
		fprintf(fp, "%d,%d\n", ta->task_id, ta->user_id);
	}
	fclose(fp);
}
/* save data of users, tasks and task_assignement to files.*/
int commit_data_to_file(){
	commit_users_to_file();
	commit_tasks_to_file();
	commit_task_assignment_to_file();
	printf("info data commited to files.\n");
}
/* read data from files */
int load_data(){
	load_users();
	load_tasks();
	load_task_assignment();
	printf("debug completed running load data for all entities");
}
/*after reading from scanf , newline will stay in buffer. this creates isuse when reading lines using
getline(). so we need to remove the newline stayed in buffer right after scanf is used.*/
int flush_new_line()
{
	char *temp = NULL;
	size_t len;
	getline(&temp, &len, stdin);
	return 0;
}
int main(){
	int user_id;
	char* user_name;
	char* user_designation;
	int task_id;
	char *task_name;
	char *task_description;
	char *task_deadline;
	char exit;
	char* options = "1. Add User \n2. Read User\n3. Update User\n4. DeleteUser\n5. Add Task\n6. Read Task\n7. Update Task\n8. Delete Task\n9.Add Task Assignment\n10.COMMIT TO FILE\n0.Exit\n";
	// testv();
	load_data();
	char *line = NULL;
	size_t len;
	while(1){
		int choice;
		printf("%s", options);
		printf("select option: ");
		scanf("%d", &choice);
		switch(choice){
			case 1: // Add User
				printf("Enter user_id: ");
				scanf("%d", &user_id);
				flush_new_line(); // above scanf leaves \n in buffer. remove \n from buffer.
				line = NULL; // reset to null. getline allocates memory automatically.
				printf("Enter user_name: ");
				getline(&line, &len, stdin);
				user_name = remove_trailing_newline(line);
				line = NULL;
				printf("Enter user_designation: ");
				getline(&line, &len, stdin);
				user_designation = remove_trailing_newline(line);
				add_user(user_id, user_name, user_designation);
				free(user_name);
				free(user_designation);
				break;
			case 2: // Read User
				printf("Enter user_id: ");
				scanf("%d", &user_id);
				flush_new_line();
				User *u = find_user(user_id);
				if (u == NULL)
				{
					printf("User with id %d not found\n", user_id);
				}
				else{
					printf("User found.\n");
					printf("user_name: %s\nuser_designation:%s\n", u->user_name, u->user_designation);
				}
				break;
			case 3: // Update User
				printf("Enter user_id: ");
				scanf("%d", &user_id);
				flush_new_line();
				line = NULL; 
				printf("Enter user_name: ");
				getline(&line, &len, stdin);
				user_name = remove_trailing_newline(line);
				line = NULL; 
				printf("Enter user_designation: ");
				getline(&line, &len, stdin);
				user_designation = remove_trailing_newline(line);
				update_user(user_id, user_name, user_designation);
				free(user_name);
				free(user_designation);
				break;
			case 4: // Delete User
				printf("Enter user_id: ");
				scanf("%d", &user_id);
				flush_new_line();
				delete_user(user_id);
				break;
			case 5: // Add Task
				printf("Enter task_id: ");
				scanf("%d", &task_id);
				flush_new_line(); // above scanf leaves \n in buffer. remove \n from buffer.
				line = NULL; // reset to null. getline allocates memory automatically.
				printf("Enter task_name: ");
				getline(&line, &len, stdin);
				task_name = remove_trailing_newline(line);
				line = NULL;
				printf("Enter task_description: ");
				getline(&line, &len, stdin);
				task_description = remove_trailing_newline(line);
				line = NULL;
				printf("Enter task_deadline: ");
				getline(&line, &len, stdin);
				task_deadline = remove_trailing_newline(line);
				add_task(task_id, task_name, task_description, task_deadline);
				free(task_name);
				free(task_description);
				free(task_deadline);
				break;
			case 6: // Read Task
				printf("Enter task_id: ");
				scanf("%d", &task_id);
				flush_new_line();
				Task *t = find_task(task_id);
				if (t == NULL)
				{
					printf("Task with id %d not found\n", task_id);
				}
				else{
					printf("Task found.\n");
					printf("task_name: %s\ntask_description:%s\ntask_deadline: %s\n", t->task_name, t->task_description, t->task_deadline);
				}
				break;
			case 7:	// Update Task
				printf("Enter task_id: ");
				scanf("%d", &task_id);
				flush_new_line(); // above scanf leaves \n in buffer. remove \n from buffer.
				line = NULL; // reset to null. getline allocates memory automatically.
				printf("Enter task_name: ");
				getline(&line, &len, stdin);
				task_name = remove_trailing_newline(line);
				line = NULL;
				printf("Enter task_description: ");
				getline(&line, &len, stdin);
				task_description = remove_trailing_newline(line);
				line = NULL;
				printf("Enter task_deadline: ");
				getline(&line, &len, stdin);
				task_deadline = remove_trailing_newline(line);
				update_task(task_id, task_name, task_description, task_deadline);
				free(task_name);
				free(task_description);
				free(task_deadline);
				break;
			case 8: // Delete task
				printf("Enter task_id: ");
				scanf("%d", &task_id);
				flush_new_line();
				delete_task(task_id);
				break;
			case 9: // Add task assignment
				user_id = 0;
				printf("Enter task_id: ");
				scanf("%d", &task_id);
				// flush_new_line();
				printf("Enter user_id: ");
				scanf("%d", &user_id);
				// flush_new_line();
				add_task_assignment(task_id, user_id);
				break;
			case 10: // commit to file.
				commit_data_to_file();
				break;
			case 0:
				printf("Make Sure you have saved changes to file. Any unsaved changes will be cleared.\n");
				printf("Do you want to exit? (Y/N): ");
				scanf(" %c", &exit);
				flush_new_line();
				while(exit != 'Y' && exit!= 'y' && exit != 'N' && exit!= 'n')
				{
					printf("invalid option\n");
					printf("Do you want to exit? (Y/N): ");
					scanf(" %c", &exit);
					flush_new_line();
				}
				if(exit == 'Y' || exit == 'y')
				{
					return 0;
				}
				break;
			default:
				printf("Invalid Option\n");
		}
	}
}
