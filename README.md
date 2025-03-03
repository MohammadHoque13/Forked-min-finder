# Forked-min-finder
#include <iostream>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <cstdlib>
#include <ctime>
#include <climits>

using namespace std;

int findMin(int arr[], int start, int end) {
    int minVal = INT_MAX;
    for (int i = start; i < end; ++i) {
        if (arr[i] < minVal) {
            minVal = arr[i];
        }
    }
    return minVal;
}

int main() {
    int arr[20];
    srand(time(0));

  cout << "Array Elements: ";
    for (int i = 0; i < 20; i++) {
        arr[i] = rand() % 100 + 1;
        cout << arr[i] << " ";
    }
    cout << endl;

   int pipefd[2];
    if (pipe(pipefd) == -1) {
        cerr << "Pipe failed!" << endl;
        return 1;
    }

  pid_t pid = fork();

   if (pid < 0) {
        cerr << "Fork failed!" << endl;
        return 1;
    } 
    else if (pid == 0) { 
        close(pipefd[0]); 
        int minChild = findMin(arr, 10, 20);
        write(pipefd[1], &minChild, sizeof(minChild));
        close(pipefd[1]); 
        cout << "Child Process ID: " << getpid() << " | Min in second half: " << minChild << endl;
    } 
    else { 
        close(pipefd[1]); 
        int minParent = findMin(arr, 0, 10);
        int minChild;
        read(pipefd[0], &minChild, sizeof(minChild));
        close(pipefd[0]); 
        wait(NULL); 

   cout << "Parent Process ID: " << getpid() << " | Min in first half: " << minParent << endl;
        cout << "Overall Minimum: " << min(minParent, minChild) << endl;
    }

   return 0;
}
