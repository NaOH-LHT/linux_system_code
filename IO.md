I/O: input & output,是一切实现的基础

-----------------------

stdio 标准IO（优先使用）

sysio系统调用IO（文件IO）

——————

stdio: FILE类型贯穿始终

*fopen() , fclose() :文件的读写

（堆区，fopen()->malloc,fclose()->free）

fgetc(),fputc()fgets(),fputs(),fread(),fwrite():字符串的读写

printf(),scanf():

fseek(),ftell(),rewind():文件位置指针的操作

fflush();