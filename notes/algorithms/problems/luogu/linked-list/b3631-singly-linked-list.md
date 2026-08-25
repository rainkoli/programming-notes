B3631 单向链表

**方法一：数组**

```cpp
#include <iostream>

using namespace std;

const int N = 1e6 + 10;

int p[N];

int main(){
    int n;
    cin >> n;
    p[1] = 0;
    int x, y, opt;
    while (n -- ) {
        cin >> opt >> x;
        if (opt == 1) {
            cin >> y;
            p[y] = p[x];
            p[x] = y;
        } else if (opt == 2) {
            cout << p[x] << endl;
        } else {
            p[x] = p[p[x]];
        }
    }
    
    
    return 0;
}
```



**方法二：结构体**

```cpp
#include <bits/stdc++.h>

using namespace std;

struct Node{
    int value;
    Node* next;
};

Node* init() {
    Node* head = new Node;
    Node* p = new Node;
    p->value = 1;
    p->next = nullptr;
    head->next = p;
    return head;
}

void insert(Node* head, int x, int y) {
    Node* p = head->next;
    while (p != nullptr) {
        if (p->value == x) {
            Node* t = new Node;
            t->value = y;
            t->next = p->next;
            p->next = t;
            break;
        }
        p = p->next;
    }
}

int query(Node* head, int x) {
    Node* p = head->next;
    while (p->next != nullptr) {
        if (p->value == x) {
            return p->next->value;
        }
        p = p->next;
    }
    
    return 0;
}

void del(Node* head, int x) {
    Node* p = head->next;
    while (p->next != nullptr) {
        if (p->value == x) {
            p->next = p->next->next;
            return ;
        }
        p = p->next;
    }
}

int main() {
    Node* head = init();
    
    int q;
    cin >> q;
    while (q -- ) {
        int opt, x, y;
        cin >> opt >> x;
        if (opt == 1) {
            cin >> y;
            insert(head, x, y);
        } else if (opt == 2) {
            cout << query(head, x) << endl;
        } else {
            del(head, x);
        }
    }
    
    return 0;
}
```

