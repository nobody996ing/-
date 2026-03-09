# -
#include<stdio.h>
#include<graphics.h>
#include<easyx.h>
#include<time.h>

IMAGE bk;//背景图片
IMAGE role;//飞机图片
IMAGE bullet;//子弹图片
IMAGE Enemy[2];//定义两个敌机,有大有小

enum MY {
	WIDTH = 480,
	HEIGHT = 700,
	bullet_num = 15,//子弹的数量
	enemy_num=10,
	big_enemy,
	small_enemy
	//方便修改
};
struct plane {
	int x;
	int y;
	bool live;//判断飞机是否死亡
	int width;
	int height;
	int hp;//血量
	int type;//敌机类型
}player,bull[bullet_num],enemy[enemy_num];
//相当于又创建了15个同类型的结构体

void loadimg() {///图片初始化
	//记得修改字符集,改为多字符集
	loadimage(&bk, "images/images/background.png");
	loadimage(&role, "images/images/me1.png");
	//子弹图片
	loadimage(&bullet, "images/images/bullet1.png");
	//敌机图片
	loadimage(&Enemy[0], "images/images/enemy1.png");
	loadimage(&Enemy[1], "images/images/enemy2.png");
}
//生产飞机
void enemyHP(int i) {
	//大飞机与小飞机的血量要不一样
	//随机生成大小飞机
	if (rand() % 8) {
		enemy[i].type = big_enemy;
		enemy[i].hp = 3;
		enemy[i].width = 104;
		enemy[i].height = 148;
	}
	else {
		enemy[i].type = small_enemy;
		enemy[i].hp = 1;
		enemy[i].width = 52;
		enemy[i].height = 39;
	}
}

void gameinit() {
	player.x = WIDTH / 2;
	player.y = HEIGHT - 120;
	player.live = true;
	//初始化子弹
	for (int i = 0; i < bullet_num; i++) {
		bull[i].x = 0;
		bull[i].y = 0;
		bull[i].live = false;
	}
	//初始化敌机
	for (int i = 0; i < enemy_num; i++) {
		//初始化第i个敌机的存活状态
		enemy[i].live = false;
		//敌机的血量
		enemyHP(i);
	}
}
//绘制子弹
void creatbullet() {

	for (int i = 0; i < bullet_num; i++) {
		if (!bull[i].live) {
			bull[i].x = player.x + 60;//是以飞机图片的左上角开始所以说这个+60就是为了子弹能从中间发射
			bull[i].y = player.y;
			bull[i].live = true;
			break;//每生成一个子弹就跳出循环
		}

	}
}
//子弹的移动
void bulletmove() {
	for (int i = 0; i < bullet_num; i++) {
		if (bull[i].live) {
			bull[i].y -= 1;
			if (bull[i].y < 0) {
				bull[i].live = false;
			}
		}

	}
}
//生产敌机
void createEnemy() {
	//首先判断我是要在没有敌机的情况下才进行生产
	for (int i = 0; i < enemy_num; i++)  {
		if (!enemy[i].live) {
			enemyHP(i);
			enemy[i].live = true;
			//重新1生成敌机时其x的坐标要求随机,y轴要求从最上面开始
			//-60是为了防止其超出范围
			enemy[i].x = rand() % (WIDTH - 60);
			enemy[i].y = 0;
			break;
		}

	}

}
//敌机的移动
void enemymove(int speed) {
	for (int i = 0; i < enemy_num; i++) {
		//依旧要先写条件
		if (enemy[i].live) {
			enemy[i].y += speed;
			//同时要考虑边界问题,防止越界
			if (enemy[i].y > HEIGHT) {
				enemy[i].live = false;
			}
		}

	}

}
void playermove(int speed) {
	//操纵键盘,键盘事件GetAsyncKeyState非阻塞函数
	//这里的UP和DOWN指的是右边的上下左右,也可用WASD来替代
	if (GetAsyncKeyState(VK_UP)) {
		if (player.y>0) { 
			player.y -= speed; 
		}
	}
	if (GetAsyncKeyState(VK_DOWN)) {
		if (player.y +120 < HEIGHT) {
			player.y += speed;
		}
	}
	if (GetAsyncKeyState(VK_LEFT) ){
		if (player.x + 50 > 0) {
			player.x -= speed;
		}
	}
	if (GetAsyncKeyState(VK_RIGHT) ){
		if (player.x + 50 < WIDTH) {
			player.x += speed;
		}
	}
	static DWORD t1 = 0, t2 = 0;
	if (GetAsyncKeyState(VK_SPACE)&&t2-t1>50) {
		creatbullet();
		t1 = t2;     
	}
	t2 = GetTickCount();
}

void gamedraw() {
	
	//初始化
	loadimg();
	//贴图
	putimage(0, 0, &bk);
	putimage(player.x, player.y, &role, SRCPAINT);
	//绘制子弹
	for (int i = 0; i < bullet_num; i++) {
		if (bull[i].live) {
			putimage(bull[i].x, bull[i].y,&bullet, SRCPAINT);
		}

	}
	//绘制敌机
	for (int i = 0; i < enemy_num; i++) {
	//先判断是否有敌机
		if (enemy[i]. live) {
			//判断是大敌机还是小敌机,然后绘制
			if (enemy[i].type = big_enemy) {
				putimage(enemy[i].x, enemy[i].y, &Enemy[1], SRCPAINT);
			}
			else {
				putimage(enemy[i].x, enemy[i].y, &Enemy[0], SRCPAINT);
			}
		}

	}

}
//子弹打敌机
void playplance() {
	for (int i = 0; i < enemy_num; i++) {
		if (!enemy[i].live) {
			continue;
		}
		for(int k=0;k<enemy_num;k++){
			if (bull[k].x > enemy[i].x && bull[k].x < enemy[i].x + enemy[i].width&&bull[k].y>enemy[i].y&&bull[k].y<enemy[i].y+enemy[i].height) {
				bull[i].live = false;
				enemy[i].hp--;
			}
			if (enemy[i].hp <= 0) {
				enemy[i].live = false;
			}
		}
	}

}
//时间戳,目的是为了使生成的敌机不同时生成
bool Time(int ms, int id) {
	static DWORD t[10];
	if (clock()-t[id]>ms) {
		//满足条件再进行同步
		t[id] = clock();
		return true;
	}
	return false;

}
int main() {   
	initgraph(WIDTH, HEIGHT, 1);//设置游戏窗口
	
	gameinit();
	//进行循环,生成游戏界面
	while (1) {
		gamedraw();
		playermove(3);
		bulletmove();
		createEnemy();

		if (Time(500, 0)) {
			createEnemy();

			 
		}enemymove(1);
		playplance();
	}

	return 0;
}
