<img width="1920" height="1140" alt="ScreenShot_2026-11-14_172112_223" src="https://github.com/user-attachments/assets/8cbb5606-35e9-45a0-8242-465677dc3018" /><img width="1920" height="1140" alt="ScreenShot_2026-11-14_171531_689" src="https://github.com/user-attachments/assets/26061022-aecd-4db5-9ba8-9d06baddc0d0" /><img width="1920" height="1140" alt="ScreenShot_2026-11-14_171524_251" src="https://github.com/user-attachments/assets/4b21c7a0-a717-4bca-9676-252e1d8d0c6f" />#### [GDOUCTF 2023]润！
查壳发现是有壳的，还让你尝试upx，直接尝试发现失败，于是想到核查壳的标志。使用**Imhex**工具查看标志字段被改了，**改回UPX字段**，再脱壳发现成功了。
![](img/ScreenShot_2026-11-14_143724_209.png)
![](img/ScreenShot_2026-11-14_143753_897.png)
查看代码的init函数发现生成了一个**puzzle**的数组，但是点击它并没有值，可以使用**动态调试**得到值    
在moving函数里发现**wsad控制，除此之外还有un控制**。目前不清楚un是什么，但能确定是个迷宫类问题
![](img/ScreenShot_2026-11-14_144112_609.png)
![](img/ScreenShot_2026-11-14_144213_185.png)
使用动态调试先获取puzzle数组，这里不需要步步调，把puzzle的**地址**记住就好，在init函数后面随便下一个断点，在**调试页面按G进行搜索**(**先复制puzzle的地址**)发现puzzle已经运行出来了。
![](img/ScreenShot_2026-11-14_150110_137.png)
这个迷宫本身不对劲，查了别的题解才发现这个迷宫是**8*8*8三维的**，不是以前遇到的二维的。前面**un每步是+-64而不是1或8**，疑似是分别控制层数上下。  
现在需要把puzzle的进行整理输出，puzzle后面的gen有好多255,253之类的数据，但我不知道如何利用这些数据**输出每一层的二维数组**，借鉴脚本输出每一层的二维数组  
gen = [127, 127, 31, 223, 223, 255, 255, 255, 255, 255,  
       241, 255, 223, 255, 255, 255, 255, 255, 197, 223,  
       223, 255, 255, 255, 255, 255, 253, 255, 255, 255,  
       255, 255, 255, 253, 253, 253, 252, 255, 255, 255,  
       255, 255, 255, 255, 254, 254, 252, 249, 255, 255,  
       255, 255, 255, 255, 255, 251, 255, 255, 255, 255,  
       255, 255, 255, 248]  
conut = 0  
map = [0]*512  
for k in range(8):  
    for i in range(8):  
        for j in range(8):  
            map[8 * i + conut + 7 - j] = (gen[int(conut / 8) + i] >> j) & 1  
    conut += 64  
    for i in range(0+k*64,64+k*64,8):  
        print(map[i:i+8])  
    print('*'*50)
![](img/ScreenShot_2026-11-14_150508_515.png)
仔细想这个脚本好像也没那么难理解。输出数组后直接**手动走一遍三维迷宫，得到路径ssddssuuwwddndduuussdussasauudd**。这个手动的过程还是比较绕的
#### [SWPUCTF 2024 秋季新生赛]动态调试
IDA打开后发现有个函数是**rc4_init**，可以判定考的函数是Rc4,后面**rc4_crypt**函数点进去看，发现和标准是一样的，key点击也能查看到，关键是不知道密文是什么 
![](img/ScreenShot_2026-11-14_154548_055.png)
最后的判断条件是**v2[j] != v1[j]**，前面的函数表明v2是用户输入的，Rc4的对象也是v2，**能做密文的只能是v1**，但是Rc4操作对象不是v1  
使用动态调试，在**rc4_crypt**下断点，尝试把v2的值修改为v1，运行后停在mov机器码处。发现**rdx, v2**这个机器码，**rdx寄存器应该存的就是v2的地址**  
![](img/ScreenShot_2026-11-14_155701_730.png)
之前先复制v1的地址，**把rdx的地址改成v1的**，再步步调试执行rc4_crypt，但是发现一直不对  
![](img/ScreenShot_2026-11-14_160105_576.png)
查了别人的题解，发现漏了一个关键信息是**r8d**，**该寄存器是存储密文的长度，而v1，v2长度并不一致，所以在修改时也必须修改长度**  
![](img/ScreenShot_2026-11-14_160823_263.png)
重新尝试，发现执行完rc4_crypt，v1的值已经改变，说明**v1被成功解密了**，把值chr变化就好了  
res = [0x4E, 0x53, 0x53, 0x43, 0x54, 0x46, 0x7B, 0x30, 0x64, 0x36,  
       0x66, 0x39, 0x30, 0x61, 0x63, 0x2D, 0x34, 0x62, 0x35, 0x65,  
       0x2D, 0x34, 0x65, 0x66, 0x62, 0x2D, 0x38, 0x35, 0x30, 0x32,  
       0x2D, 0x36, 0x33, 0x34, 0x39, 0x63, 0x66, 0x37, 0x39, 0x38,  
       0x66, 0x32, 0x65, 0x7D]  
for i in res:  
    print(chr(i), end='')
#### [SWPUCTF 2022 新生赛]贪吃蛇
IDA打开后，对main函数进行反编译，伪代码里有gameStartMenu，gameMainLoop，先分别查看一下两个函数，后一个函数应该是控制游戏进程，判断结果，输出结果等    
查找了半天发现_t_flag疑似是待输出的内容，而前面的校验语句有点复杂。关键是如何通过动态调试跳过校验或者是修改逻辑
![](img/ScreenShot_2026-11-14_163131_960.png)
先直接运行观察逻辑，发现jz short loc_401BF3的**跳转语句**，**左边是isflag的输出语句，右边是gameover的语句**，在**jz short loc_401BF3**处下断点，观察运行  
![](img/ScreenShot_2026-11-14_164406_031.png)
运行直接让蛇死掉，发现程序**执行到jz short loc_401BF3停下了**，此时发现**闪烁的是右边的线段，说明前面的校验结果生效了**
![](img/ScreenShot_2026-11-14_165151_377.png)
此时可以直接**修改ZF的值**来修改校验结果，此时的ZF为1(校验结果生效)，改成0，会发现**闪烁的是左边的线段，说明修改成功，校验结果已经不生效**。再点运行会输出flag  
![](img/ScreenShot_2026-11-14_165201_885.png)
**ZF本身是用于表示判断的结果，在断点处修改ZF等于说直接修改了前面的判断结果，使得程序原本的分支发生变化，执行的模块随之发生改变**
#### [HNCTF 2022 WEEK3]Try2debugPlusPlus
IDA打开，发现函数tea_encrypt，判断考察tea，打开该函数观察加密逻辑，发现其与标准一样，点enc发现有数组，应该对应的是密文
![](img/ScreenShot_2026-11-14_171524_251.png)
![](img/ScreenShot_2026-11-14_171531_689.png)
现在不知道的信息是**密钥**，直接使用v6的四个数据，发现是错误的，发现main函数里的 内容似乎和密钥的输出有关,尝试动态调试  
在**main**这里下断点，直接运行发现会退出，主要是**call exit**机器码使得程序退出，为了让退出指令无效，尝试**把该指令nop掉**再运行，发现还不行
![](img/ScreenShot_2026-11-14_172112_223.png)
**后面发现jz short loc_401848跳转语句也是无效的，需要把该指令也nop掉，把nop后的一些无效的db也一起nop掉才行**
![](img/ScreenShot_2026-11-14_173731_207.png)
最后处理完在运行，输出了Key:0x91,0x71,0x11,0xbb。这样条件就齐了，可以写对应过来解密的程序了。注意每两组密文进行解密，循环条件，循环次数，一定要写对  
#include <stdio.h>  
#include <string.h>  
#include <stdint.h>  
int main(void)  
{  
    unsigned int a1[]={0x0DAD5B6C5, 0x0CE5F717, 0x8BE8AF6B, 0x0D74C6EB4, 0x0EEB8B5A0, 0x0A07618E0, 0x1B425FD0, 0x0C0B77641, 0x0A30FA9BE,0x0CB4F5089, 0x0EBF9EC1D, 0x0F870EF3D};  
    int a2[] = {0x91,0x71,0x11,0xbb};  
    unsigned int v0,v1,i;  
    for (int k = 0; k < 12; k += 2)  
    {  
        v0 = a1[k];  
        v1 = a1[k + 1];  
        long delta = 0x79B99E37;  
        long sum = (32 * delta);  
        for (i = 0; i <= 31; ++i)  
        {  
            v1 -= (((v0 * 16) ^ (v0 >> 5)) + v0) ^ (sum + a2[(sum >> 11 ) & 3]);  
            sum -= delta;  
            v0 -= (((v1 * 16) ^ (v1 >> 5)) + v1) ^ (sum + a2[(sum & 3)]);  
        }  
        a1[k] = v0;  
        a1[k+1] = v1;  
    }  
    for (i = 0; i < 12; i++)  
    {  
        printf("%c",a1[i]);  
    }  
}
#### [WUSTCTF 2020]level4
IDA打开，反编译后发现left，right，node这些，好像考察的是二叉树，二叉树我已经忘记的差不多了，正好重新温习一遍  
type1,type2两个函数疑似是遍历输出二叉树，还有一个字符串是二叉树处理的对象。没有其他信息了。
重新查阅有关二叉树的资料，发现特征很明显  
**前序的顺序是:根左右**  
**中序的顺序是:左根右**  
**后序的顺序是:左右根**  
函数的putchar疑似是打印结点，也就是打印根，所以据此判断**type1是中序**，**type2是后序**。但是还不知道输出的结果  
尝试运行文件发现报错，提示**指定的可执行文件不是此操作系统平台的有效应用程序**。发现不对劲，查壳发现这个文件是Linux文件而非windows  
为了完整复现，花了很长时间新建虚拟机，在虚拟机里打开文件并执行它，**输出了type1，type2函数的内容**，已知中序遍历的输出，后序遍历的输出，求前序遍历的输出。  
据此写解题代码  * left，* right是左子树，右子树的结点，inStart，int inEnd是中序遍历的起始，最大值，postStart，int postEnd是后序遍历的起始，最大值  
if (inorder[i] == postorder[postEnd])是判断最终的根节点的位置，root->left，root->right就是构建左右子树  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
struct TreeNode   
{
    char val;
    struct TreeNode* left;  
    struct TreeNode* right;  
};
struct TreeNode* buildTree(char* inorder, int inStart, int inEnd, char* postorder, int postStart, int postEnd)   
{  
    if (inStart > inEnd || postStart > postEnd)  
    {  
        return NULL;  
    }  
    struct TreeNode* root = (struct TreeNode*)malloc(sizeof(struct TreeNode));  
    root->val = postorder[postEnd];  
    int i;  
    for (i = inStart; i <= inEnd; i++)  
    {  
        if (inorder[i] == postorder[postEnd])  
        {  
            break;  
        }  
    }  
    root->left = buildTree(inorder, inStart, i - 1, postorder, postStart, postStart + i - inStart - 1);  
    root->right = buildTree(inorder, i + 1, inEnd, postorder, postStart + i - inStart, postEnd - 1);  
    return root;  
}  
void preOrder(struct TreeNode* root)   
{  
    if (!root)   
    {  
        return;  
    }  
    printf("%c", root->val);  
    preOrder(root->left);  
    preOrder(root->right);  
}  
int main()  
{  
    char inorderStr[] = "2f0t02T{hcsiI_SwA__r7Ee}";  
    char postorderStr[] = "20f0Th{2tsIS_icArE}e7__w";  
    int n = strlen(inorderStr);  
    struct TreeNode* root = buildTree(inorderStr, 0, n - 1, postorderStr, 0, n - 1);  
    printf("前序遍历: ");  
    preOrder(root);  
    return 0;  
}  
