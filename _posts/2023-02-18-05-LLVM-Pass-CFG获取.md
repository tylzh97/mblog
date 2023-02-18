---
layout: post
title: 使用 LLVM Pass 生成控制流图
categories: LLVM
description: 使用 LLVM Pass 生成控制流图
keywords: LLVM
---


### 获取控制流结构

本质上, 获取函数中的控制流, 只需要找到所有的跳转语句即可. 在 IR 层面, 跳转语句包含 `br`, `switch` . 用于提取控制流的 Pass 代码, 即 `Hello.cpp` 内容如下:

```cpp
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/InstrTypes.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Pass.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
// llvm::TypeFinder
#include "llvm/IR/TypeFinder.h"
// llvm::formatv
#include "llvm/Support/FormatVariadic.h"

using namespace llvm;

// 将 LLVM::Value 转换为 std::String
std::string v2s(llvm::BasicBlock* v) {
  std::string str;
  llvm::raw_string_ostream S(str);
  S << *v;
  return str;
}

// 替换字符串中的全部字串为新串, 类似 python 中的 str.replace
std::string replaceAll(std::string str,
                       std::string old_value,
                       std::string new_value) {
  size_t start_pos = 0;
  while ((start_pos = str.find(old_value, start_pos)) != std::string::npos) {
    str.replace(start_pos, old_value.length(), new_value);
    start_pos += new_value.length();
  }
  return str;
}

// 将 llvm::Value 转换为 std::String, 同时转换为 CFG 可读模式
std::string v2s_c(llvm::BasicBlock* v) {
  std::string str;
  llvm::raw_string_ostream S(str);
  S << *v;
  std::string ret1 = replaceAll(str, std::string("\n"), std::string("\\l"));
  std::string ret2 = replaceAll(ret1, std::string("\""), std::string("\\\""));
  return ret2;
}

namespace {
struct SkeletonPass : public FunctionPass {
  static char ID;
  SkeletonPass() : FunctionPass(ID) {}

  // 析构函数
  ~SkeletonPass() {}

  virtual bool runOnFunction(Function& F) {
    for (auto& B : F) {
      for (auto& I : B) {
        // 判断当前的指令是否为 br 指令
        if (isa<BranchInst>(&I)) {
          BranchInst* branch = cast<BranchInst>(&I);
          // 后续基本块数目
          int num_of_successors = 0;
          // 迭代后继基本块
          for (auto* sb : branch->successors()) {
            errs() << num_of_successors << ":\t" << F.getName() << " ---- "
                   << "Relationship between " << B.getName() << " and "
                   << sb->getName() << " EdgeType=> ";
            // 无条件分支
            if (branch->isUnconditional()) {
              errs() << "Unconditional Edge"
                     << "\n";
              // 有条件分支
            } else {
              errs() << "Conditional Edge "
                     << "\n";
            }
            ++num_of_successors;
          }
        }
        // 判断是否为 Switch 语句
        if (isa<SwitchInst>(&I)) {
          SwitchInst* Switch = cast<SwitchInst>(&I);
          // 迭代所有的 Switch 有值选项
          for (auto s_case : Switch->cases()) {
            BasicBlock* sb = s_case.getCaseSuccessor();
            errs() << "Switch --- case: " << *s_case.getCaseValue()
                   << " ------ " << *sb << "\n";
          }
          // 获取 Switch 的默认分支
          auto db = Switch->case_default();
          BasicBlock* default_db = db->getCaseSuccessor();
          errs() << *default_db << "<----- This is cases(default)\n";
        }
      }
    }
    return false;
  }
};
}  // namespace

char SkeletonPass::ID = 0;

static RegisterPass<SkeletonPass> X("rhpass", "Hello World Pass");

// Automatically enable the pass.
// http://adriansampson.net/blog/clangpass.html
static void registerSkeletonPass(const PassManagerBuilder&,
                                 legacy::PassManagerBase& PM) {
  PM.add(new SkeletonPass());
}
static RegisterStandardPasses RegisterMyPass(
    PassManagerBuilder::EP_EarlyAsPossible,
    registerSkeletonPass);
```

 测试样本代码 `main.c` 内容如下:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TEMPLATE "Hello World!"

typedef struct School {
    char *name;
    unsigned int cost;
    char *address;
    
} School;

typedef struct Human {
    char *name1;
    char *name2;
    char *name3;
    unsigned int age;
    short educate_level;
    School* school;
} Human;


size_t my_add(size_t nn1, size_t nn2, size_t nn3) {
    return nn1 + nn2 + nn3;
}

size_t memchange(char* str) {
    for (int i=0; i<3; i++) {
        str[i+1] = 'a' + i;
    }
    if ('H' == *str || 'o' == *str) {
        *str = '~';
    } else {
        str[1] = '+';
    }
    return 3;
}


int main(int argc, const char** argv) {
    size_t num;
    num = my_add(4,6,1);
    printf("===\n[Input %zu]: ", num);
    scanf("%zi", &num);
    printf("%lu\n", num + 4);
    char* str;
    str = (char*)malloc((sizeof(char)*num));

    Human *jack;    
    switch (num) {
        case 5:
            jack = (Human *)malloc(sizeof(Human));
            jack->age = 18;
            free(jack);
            printf(">>> Main Switch --- This is %d\n", num);
            for (int j = 0; j < 15; j++) {
                printf ("Anything...\n");
            }
            break;
        case 6:
            memchange(str);
            printf(">>> Main Switch --- This is %d\n", num);
            break;
        case 7:
            memchange(str+5);
            printf(">>> Main Switch --- This is %d\n", num);
            break;
        case 8:
            printf(">>> Main Switch --- This is %d\n", num);
            break;
        default:
            printf(">>> Main Switch --- Default Branch %d\n", num);
            break;
    }
    

    memcpy(str, TEMPLATE, sizeof(TEMPLATE));
    printf("\n>>> %p %s\n", str, str);
    memchange(str);
    printf("\n>>> %p %s\n", str, str);
    memchange(str+2);
    free(str);
    return 0;
}
```

使用如下 `Makefile` 对文件进行编译:

```makefile
CC = clang
CXX = clang++
LLVM_FLAGS = `llvm-config --cxxflags` -Wl,-znodelete -fno-rtti -fPIC --no-warnings
LLVM_LDFLAGS = `llvm-config --ldflags`
CFLAGS = -flegacy-pass-manager -g -Xclang -load -Xclang ./LLVMHello.so -Wno-format


clean:
	@rm -f *.so *.elf *.ll *.dot *.o

pass: clean
	@$(CXX) ${LLVM_FLAGS} -fexceptions -shared Hello.cpp -o LLVMHello.so ${LLVM_LDFLAGS}

example: clean pass
	@$(CC) ${CFLAGS} -c main.c -o main.o
	@$(CC) main.o -o main.elf

all: example
	@echo "Finish!"
```



### 关键 API 说明



1. 判断当前指令是否为跳转指令, 如果是则获取 `llvm::BranchInst`

```cpp
if (isa<BranchInst>(&I)) {
    BranchInst* branch = cast<BranchInst>(&I);
}
```

2. 获取 `br` 的后继节点与分支类型(有条件分支/无条件分支)

```cpp
// 获取后继节点
for (auto* sb : branch->successors()) {
    // B 当前基本块; sb 后继基本块
    errs() << F.getName() << " ---- " << B.getName() << " <--->"
           << sb->getName() << " EdgeType <===> ";
    // 判断是否为无条件分支
    if (branch->isUnconditional()) {
        errs() << "Unconditional Edge"<< "\n";
    } else {
        errs() << "Conditional Edge "<< "\n";
    }
}
```

3. 判断当前指令是否为 `switch` 指令, 如果是则获取 `llvm::SwitchInst`

```cpp
if (isa<SwitchInst>(&I)) {
    SwitchInst* Switch = cast<SwitchInst>(&I);
}
```

4. 获取 `Switch` 的条件分支与 `default` 分支

```cpp
// 迭代所有的 Switch 的 Cases
for (auto s_case : Switch->cases()) {
    BasicBlock* sb = s_case.getCaseSuccessor();
    errs() << "case: " << *s_case.getCaseValue() << " -- " << *sb << "\n";
}
// 获取 Switch 的默认分支
auto db = Switch->case_default();
BasicBlock* default_db = db->getCaseSuccessor();
errs()<< "case: " << " deuault " << " -- " << *default_db << "\n";
```























































