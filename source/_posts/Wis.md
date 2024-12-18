---
title: Wis
date: 2024-11-26 16:30:35
tags:
---

# 1 WIS测井数据文件格式

## 1.1 WIS 文件结构

### 1.1.1 文件标识

WIS文件标识从文件偏移零开始，为10个字节的字符。当前版本的标识为WIS 1.0。

### 1.1.2 文件头结构

头结构紧接文件标识。描述WIS文件的公共信息。结构定义如下：

```c
typedef struct tagWIS_HEAD		//**偏移**    **字节数**     **描述**
{
	WORD	MachineType;	//		0        	 2            机器类型=1 为PC； =2为SUN； =3为IBM； =4为HP。
	WORD	MaxObjectNumber;//		2       	 2        允许记录的最大对象数。缺省为512个，该值可以在文件产生时给出。
	WORD	ObjectNumber;	//		4	         2            当前记录的对象总数（包括删除和抛弃的对象）。
	WORD	BlockLen;		//		6    		2  		块长。WIS文件对象占用的磁盘空间以块为单位，该值指示每一数据块的字节数。
	DWORD	EntryOffset;	//		8        	 4            对象入口记录从文件开始的偏移量。
	DWORD	DataOffset;		//		12			4            对象数据记录从文件开始的偏移量。
	DWORD	FileSize;		//		16      	 4            WIS文件的字节数大小。
	DWORD	TimeCreate;		//		20			4            WIS文件产生的时间。
	char	Reserved[32];	//		24			32           保留字节。
}WIS_HEAD;
```

### 1.1.3 对象入口

对象入口描述每个对象的公共信息，开始位置由头结构给出。每个对象的描述信息前后相连。结构定义如下：

```c
typedef struct tagWIS_OBJECT_ENTRY	 //		**偏移**   **字节数**       **描述**
{
    char	Name[16];				//		0		16            对象的名称，以零结尾的字符串。
    long	Status;					//		16		 4             对象的状态：=0为正常； =1为抛弃； =2为删除。
    short	Attribute;				//		20		 2         对象的主属性：=1为通道对象； =2为表对象； =3为流对象。
    short	SubAttribute;			//		22		 2             对象的子属性，描述对应主属性的子属性。
    DWORD	Position;				//		24		 4             对象数据体从文件开始处的偏移量。
    DWORD	BlockNum;				//		28		 4             对象数据体占用磁盘的块数。
    DWORD	TimeCreate;				//		32		 4             对象产生的时间。
    DWORD	TimeWrite;				//		36		 4             对象最近修改的时间
    char	Reserved[32];			//		40		32            保留字节。
}WIS_OBJECT_ENTRY;
```

### **1.1.4** 对象数据体

对象数据体记录各个对象的具体特性及数据。根据不同的主属性分三种类型。对象数据体在WIS文件中的位置由对象入口指定。

### **1.1.5** 通道对象

通道对象用来存放采集和计算结果数据(如测井曲线)。分为通道信息和通道数据两部分。

WIS文件将在一定时空内对某一采集或计算的物理信息数据集统称为通道数据。通道信息描述通道数据的存放形式，分为基本信息和维信息，基本信息描述信息的基本物理含义，维信息描述信息的时空特性，可以等间隔(连续)或非等间隔(离散)。最大允许有四维信息，通道信息共占用一个块空间，结构定义如下：

```c
typedef struct tagWIS_CHANNLE		//	偏移		字节数		描述
{
    char	Unit[8];				//  0	 		8	    对象的单位，以零结尾的字符串。
    char	AliasName[16];			// 	8			16		对象的别名，以零结尾的字符串。
    char	AliasUnit[16];			//  24			16		单位的别名，以零结尾的字符串。
    WORD	RepCode;				// 40			2		对象数据类型，参见
    WORD	CodeLen;				// 42			2		数据类型的长度
    float	MinVal;					// 44			4		对象的最小值(测井曲线缺省左刻度值)
    float	MaxVal;					// 48			4		对象的最大值(测井曲线缺省右刻度值)。
    WORD	Reserved;				// 52			2		保留字节
    WORD	NumOfDimension;			// 54			 2		 对象维信息数。
    WIS_CHANNEL_DIMENSION DimInfo[4];//	56			 4*56	   对象维信息。
}WIS_CHANNEL;
```

通道维信息结构定义如下：

```C
typedef struct tagWIS_CHANNLE_DIMENSION // 	偏移		字节数	   描述
{
    char	Name[8];				//		0		8		维的名称，以零结尾的字符串。
    char	Unit[8];				//		8		8		维的单位，以零结尾的字符串。
    char	AliasName[16];			//		16		16		维的别名，以零结尾的字符串。
    float	StartVal;				//		32		4		维的开始值。
    float	Delta;					//		36		4		维的采集或计算增量。对于离散数据，该值为0，数据中记录该维的值。
    DWORD	Samples;				//		40		4		维的数据采样点数。如果该值为0，采样点数为可变值，数据中记录该值。对于第一维数据，该值不能为0。
    DWORD	MaxSamples;				//		44		4		维的数据采样最大点数。该值仅当采样点数信息为0（可变采样点）时有效，该维信息在数据中所占用的字节数通过该值计算。
    DWORD	Size;					//		48   	4		该维上每一采样点所占用的字节数。
    WORD	RepCode;				//		52		2		维的数据类型，参见
    WORD	Reserved;				//		54		2		保留字节。
}WIS_CHANNEL_DIMENSION;
```

通道数据从通道描述信息的下一块开始。

下面为一个包含深度和时间维的物理信息数据体的存放顺序。第一维为深度，第二维为时间。

**[A1]+[N2]+[B1]+X1+[B2]+X2+**···**+[BN]+XN+**

**[A2]+[N2]+[B1]+X1+[B2]+X2+**···**+[BN]+XN+**

···

···

**[AN]+[N2]+[B1]+X1+[B2]+X2+**···**+[BN]+XN**

**其中：**

A1，A2，··· ，AN代表深度值，当深度维信息结构中的Delta为零时，记录此值。

N2代表当前深度点上的时间采样点数，当时间维信息结构中的采样点数为零时，记录此值。

B1，B2，··· ，BN代表时间值，当时间维信息结构中的Delta为零时，记录此值。

X1，X2，··· ，N代表物理信息的值。

### **1.1.6** 表对象

表对象用来存放二维表数据，分为表信息和表数据体两个部分。表信息由不同的表项组成，每一表项称为字段。表信息结构定义如下：

```c
typedef struct tagWIS_TABLE	//偏移	字节数		描述
{
    DWORD	RecordCount;	//0			4	表的记录数。
    DWORD	FieldCount;		//4			4	表的字段数。
    WIS_TABLE_FIELD *pField;//8			4	指向字段信息结构的指针。
}WIS_TABLE;
```

字段信息结构定义如下：

```c
typedef struct tagWIS_TABLE_FIELD	//偏移	字节数			描述
{
    char	Name[32];				//0		32           	字段的名称，以零结尾的字符串。
    WORD	RepCode;				//32	2            	字段值的浮点类型，参见。
    WORD	Length;					//34	2            	字段值的长度。
    DWORD	Reserved;				//36	4				保留字节。
}WIS_TABLE_FIELD;
```

表数据体(记录)从表信息记录的下一块开始。

### **1.1.7** 流对象

流对象用来存放二进制数据块。开始为4个字节的无符号长整形数，代表数据流的长度。接着为该流的二进制值。
