# snake-
#include <cstdlib>
#include <iostream>
#include <windows.h>
#include <conio.h>
#include <ctime> // 需要 time(0)
using namespace std;

char Map[105][105];
int Lx, Ly;
int snakeX[100], snakeY[100];
int len = 3;

void HideCursor() {
    CONSOLE_CURSOR_INFO cursor;
    cursor.bVisible = FALSE;
    cursor.dwSize = 1;
    SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &cursor);
}

void Menu() {
    cout << "贪吃蛇小游戏" << endl;
    cout << "1、游戏开始" << endl;
    cout << "2、游戏说明" << endl;
    cout << "3、退出游戏" << endl;
    cout << "请输入数字：" << endl;
}

void Description() {
    ::system("cls");
    cout << "游戏说明" << endl;
    cout << "1、wasd控制蛇上下左右移动" << endl;
    cout << "2、按空格键暂停/开始游戏" << endl;
    cout << "3、按b键加速" << endl;
    cout << "4、按空格键返回菜单" << endl;
    while (1) {
        if (_kbhit()) {
            char key = _getch();
            if (key == 32) { // 空格
                ::system("cls");
                return; // 直接返回，由 main 里的 while 自动调 Menu
            }
        }
    }
}
int gamespeed = 150;
void Game() {
    HideCursor();
    int score = 0;
    len = 3;

    // 1. 初始化地图
    for (int i = 1; i <= Ly; i++) {
        for (int j = 1; j <= Lx; j++) {
            Map[i][j] = ' ';
        }
    }
    for (int i = 0; i <= Ly + 1; i++) { Map[i][0] = Map[i][Lx + 1] = '#'; }
    for (int j = 0; j <= Lx + 1; j++) { Map[0][j] = Map[Ly + 1][j] = '#'; }

    // 2. 初始化蛇（放在地图中间）
    snakeX[0] = Lx / 2;     snakeY[0] = Ly / 2;     // 头
    snakeX[1] = Lx / 2 - 1; snakeY[1] = Ly / 2;     // 身
    snakeX[2] = Lx / 2 - 2; snakeY[2] = Ly / 2;     // 尾

    // 3. 初始化食物
    srand(time(0)); // 随机种子
    int foodX = rand() % Lx + 1;
    int foodY = rand() % Ly + 1;
    for (int i = len - 1; i >=0; i--) {
        if (foodX == snakeX[i] && foodY ==snakeY[i]) {
            foodX = rand() % Lx + 1;
            foodY = rand() % Ly + 1;
        }
    }
    char dir = 'd'; // ?? 必须给 key 一个初始值，默认向右

    // 4. 游戏主循环
    while (true) {
        // 4.1 接收按键
        if (_kbhit()) {
            char ch = _getch();
            if (ch == 'w' || ch == 'a' || ch == 's' || ch == 'd') {
                dir = ch;
            }
            else if (ch == 'v') {
                gamespeed = 100;
            }
            else if (ch == 'b') {
                gamespeed = 300;
            }
        }

        // 4.2 计算新蛇头坐标
        int newHeadX = snakeX[0], newHeadY = snakeY[0];
        switch (dir) {
        case 'w': newHeadY--; break;
        case 's': newHeadY++; break;
        case 'a': newHeadX--; break;
        case 'd': newHeadX++; break;
        default: break; // 其他按键，直接break忽略
        }
        
        // 4.3 撞墙检测（?? 注意，这里不包含移动逻辑！）
        bool isdead = false;
        if (newHeadX <= 0 || newHeadX > Lx || newHeadY <= 0 || newHeadY > Ly) {
            isdead = true;
        }
        for (int i = 1; i < len; i++) {
            if (newHeadX == snakeX[i] && newHeadY == snakeY[i])
                isdead = true;
        }
        if (isdead) {
            system("cls");
            cout << "游戏结束" << endl;
            cout << "score=" << score << endl;
            system("pause");
            system("cls");
            return;
        }

        // 4.4 蛇身移动（从尾到头覆盖）
        for (int i = len - 1; i > 0; i--) {
            snakeX[i] = snakeX[i - 1];
            snakeY[i] = snakeY[i - 1];
        }
        snakeX[0] = newHeadX;
        snakeY[0] = newHeadY;

        // 4.5 吃食物检测
        if (snakeX[0] == foodX && snakeY[0] == foodY) {
            len++;
            score = score + 5;
            foodX = rand() % Lx + 1;
            foodY = rand() % Ly + 1;

        }

        // 4.6 把蛇和食物画到 Map 上（?? 坐标顺序：先行后列！）
        // 1. 清空地图内部（擦掉上一帧的残影）
        for (int i = 1; i <= Ly; i++) {
            for (int j = 1; j <= Lx; j++) {
                Map[i][j] = ' ';
            }
        }

        // 2. 重新画边界（防止边界被清空覆盖掉）
        for (int i = 0; i <= Ly + 1; i++) { Map[i][0] = Map[i][Lx + 1] = '#'; }
        for (int j = 0; j <= Lx + 1; j++) { Map[0][j] = Map[Ly + 1][j] = '#'; }

        Map[foodY][foodX] = '*';
        for (int i = 0; i < len; i++) {
            Map[snakeY[i]][snakeX[i]] = 'O';
        }

        // 4.7 渲染画面
        ::system("cls");
        for (int i = 0; i <= Ly + 1; i++) {
            for (int j = 0; j <= Lx + 1; j++) {
                cout << Map[i][j];
            }
            cout << endl;
        }
        cout << "贪吃蛇：1、wasd为方向键 2、空格暂停 3、v加速，b减速" << endl;
        cout << "score:" << score << endl;

        // 4.8 控制速度
        Sleep(gamespeed);
    }
}

int main() {
    while (true) {
        Menu();
        int Num;
        cin >> Num;
        switch (Num) {
        case 1: {
            ::system("cls");
            cout << "地图默认大小为20*40,若想使用默认地图输入1，想自定义地图输入2：" << endl;
            int i; cin >> i;
            if (i != 1) {
                cout << "地图大小长不能超过60，宽不能超过90，请输入宽：";
                int wide; cin >> wide;
                Lx = (wide < 10 || wide > 90) ? 40 : wide;
                cout << "请输入长: ";
                int height; cin >> height;
                Ly = (height < 10 || height > 60) ? 20 : height;
            }
            else {
                Lx = 40, Ly = 20;
            }
            Game();
            break;
        }
        case 2:
            Description();
            break;
        case 3:
            return 0;
        default:
            cout << "输入有误，请再次输入:" << endl;
            break;
        }
    }
    return 0;
}
