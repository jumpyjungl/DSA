# DSA
This C++ program is a console-based Event Management System. It allows administrators or users to manage events by adding, updating, deleting, searching, and sorting them. 

#include <iostream>
#include <fstream>
#include <string>
#include <iomanip>
#include <queue>
#include <vector>
#include <algorithm>
using namespace std;

struct Event {
    int id;
    string name;
    string type;
    string date;
    Event* next;
};

class EventList {
private:
    Event* head;

public:
    EventList() { head = nullptr; }

    void addEvent(int id, string name, string type, string date) {
        Event* newEvent = new Event{id, name, type, date, nullptr};
        if (!head) head = newEvent;
        else {
            Event* temp = head;
            while (temp->next) temp = temp->next;
            temp->next = newEvent;
        }
        saveToFile();
    }

    void updateEvent(int id, string name, string type, string date) {
        Event* temp = head;
        while (temp) {
            if (temp->id == id) {
                temp->name = name;
                temp->type = type;
                temp->date = date;
                saveToFile();
                return;
            }
            temp = temp->next;
        }
    }

    void deleteEvent(int id) {
        if (!head) return;
        if (head->id == id) {
            Event* toDelete = head;
            head = head->next;
            delete toDelete;
            saveToFile();
            return;
        }
        Event* temp = head;
        while (temp->next && temp->next->id != id)
            temp = temp->next;
        if (temp->next) {
            Event* toDelete = temp->next;
            temp->next = toDelete->next;
            delete toDelete;
            saveToFile();
        }
    }

    void displayEvents() {
        Event* temp = head;
        cout << left << setw(5) << "ID" << setw(20) << "Name" << setw(15) << "Type" << setw(15) << "Date" << endl;
        while (temp) {
            cout << left << setw(5) << temp->id << setw(20) << temp->name << setw(15) << temp->type << setw(15) << temp->date << endl;
            temp = temp->next;
        }
    }

    void searchEventLinear(string key) {
        Event* temp = head;
        while (temp) {
            if (temp->name == key || temp->type == key || temp->date == key) {
                cout << "Found: " << temp->id << ", " << temp->name << ", " << temp->type << ", " << temp->date << endl;
                return;
            }
            temp = temp->next;
        }
        cout << "Event not found.\n";
    }

    void saveToFile() {
        ofstream file("events.txt");
        Event* temp = head;
        while (temp) {
            file << temp->id << "," << temp->name << "," << temp->type << "," << temp->date << endl;
            temp = temp->next;
        }
        file.close();
    }

    void loadFromFile() {
        ifstream file("events.txt");
        string line;
        while (getline(file, line)) {
            int id;
            string name, type, date;
            size_t pos1 = line.find(",");
            size_t pos2 = line.find(",", pos1 + 1);
            size_t pos3 = line.find(",", pos2 + 1);
            id = stoi(line.substr(0, pos1));
            name = line.substr(pos1 + 1, pos2 - pos1 - 1);
            type = line.substr(pos2 + 1, pos3 - pos2 - 1);
            date = line.substr(pos3 + 1);
            addEvent(id, name, type, date);
        }
        file.close();
    }

    vector<Event*> toVector() {
        vector<Event*> v;
        Event* temp = head;
        while (temp) {
            v.push_back(temp);
            temp = temp->next;
        }
        return v;
    }

    void mergeSort(vector<Event*>& v, int left, int right) {
        if (left >= right) return;
        int mid = left + (right - left) / 2;
        mergeSort(v, left, mid);
        mergeSort(v, mid + 1, right);
        merge(v, left, mid, right);
    }

    void merge(vector<Event*>& v, int left, int mid, int right) {
        vector<Event*> temp;
        int i = left, j = mid + 1;
        while (i <= mid && j <= right) {
            if (v[i]->date < v[j]->date)
                temp.push_back(v[i++]);
            else
                temp.push_back(v[j++]);
        }
        while (i <= mid) temp.push_back(v[i++]);
        while (j <= right) temp.push_back(v[j++]);
        for (int k = left; k <= right; ++k) v[k] = temp[k - left];
    }

    void sortByDate() {
        vector<Event*> v = toVector();
        mergeSort(v, 0, v.size() - 1);
        head = nullptr;
        for (Event* e : v) {
            e->next = nullptr;
            if (!head) head = e;
            else {
                Event* temp = head;
                while (temp->next) temp = temp->next;
                temp->next = e;
            }
        }
    }

    void binarySearchByID(int id) {
        vector<Event*> v = toVector();
        sort(v.begin(), v.end(), [](Event* a, Event* b) { return a->id < b->id; });
        int left = 0, right = v.size() - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (v[mid]->id == id) {
                cout << "Found: " << v[mid]->id << ", " << v[mid]->name << ", " << v[mid]->type << ", " << v[mid]->date << endl;
                return;
            } else if (v[mid]->id < id) left = mid + 1;
            else right = mid - 1;
        }
        cout << "Event ID not found.\n";
    }
};

class UserSystem {
private:
    queue<string> userQueue;

public:
    void registerUser(string username) {
        userQueue.push(username);
        cout << username << " registered for event successfully.\n";
    }

    void showRegisteredUsers() {
        queue<string> copy = userQueue;
        cout << "Registered Users:\n";
        while (!copy.empty()) {
            cout << "- " << copy.front() << endl;
            copy.pop();
        }
    }
};

int main() {
    EventList el;
    el.loadFromFile();
    UserSystem us;

    int n;
    cout << "How many events do you want to add? ";
    cin >> n;
    cin.ignore();

    for (int i = 0; i < n; ++i) {
        int id;
        string name, type, date;

        cout << "\nEnter Event ID: ";
        cin >> id;
        cin.ignore();
        cout << "Enter Event Name: ";
        getline(cin, name);
        cout << "Enter Event Type: ";
        getline(cin, type);
        cout << "Enter Event Date (YYYY-MM-DD): ";
        getline(cin, date);

        el.addEvent(id, name, type, date);
    }

    cout << "\nAll Events:\n";
    el.displayEvents();

    cout << "\nSorted by Date:\n";
    el.sortByDate();
    el.displayEvents();

    string searchName;
    cout << "\nEnter name/type/date to search: ";
    getline(cin, searchName);
    el.searchEventLinear(searchName);

    int searchID;
    cout << "\nEnter Event ID to search with Binary Search: ";
    cin >> searchID;
    el.binarySearchByID(searchID);

    cin.ignore();
    string username;
    cout << "\nEnter your name to register for event: ";
    getline(cin, username);
    us.registerUser(username);

    us.showRegisteredUsers();

    return 0;
}
