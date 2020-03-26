# BA08-Interface-Docs

## Software Environment
Operating in the Windows XP/XPE (Chinese/English) or Windows 2000 (Chinese/English) system.
1. Based on the new Dunite subject to version V3.3.0;
2. Header files required: BillDepositDev.h, BillDepositDevDef.h and BillDepositDevStruct.h;
3. Necessary library files: Debug: BillDepositDevDllD.dll, BillDepositDevDllD.lib Release: BillDepositDevDll.dll and BillDepositDevDll.lib
4. Necessary configuration files: GRGDTATM_CommCfg.ini BillDepositDevCfg.ini

Note: If Dunite works with a registry, add the associated configuration items in the GRGDTATM_CommCfg.ini file into the registry


## Hardware Environment
1. Industrial PC: above PIII 800;
2. BR15, BA15, BA08, Bim2020 and Mei;
3. Power supply;
4. Cables;
Compatible hardware version: none
Incompatible with earlier versions of the dynamic library for a driver: none


## General Description
Status codes fall into physical and logical codes.

* A physical code refers to the code actually returned for a device, showing a specific and real status of the device.

* A logical code refers to the code returned upon analysis and classification of physical codes, showing the status of the same nature for a kind of device.

 Physical and logical codes are in the following relation: one logical code may correspond to several or one physical code.

 When using the code returned for the status, the upper software will take the logical code for reference.

 The error codes returned from a module involve the errors for communication and device. A macro definition indicates a status code for successful operation when beginning with S_, a code for error when beginning with E, and a code for warning when beginning with W_.



 |       Macro Definition       | Error Code 	|                Description            |
|:----------------------------:	|:----------:	|:----------------------------------------:	|
| W_UNSUPPORT_COMMAND          	| 998        	| Command is not supported                 	|
| W_BA15DEV_RECEIVE_REAL_NOTES 	| 2100       	| The module received real notes (prompt). 	|
| W_BA15DEV_RECEIVE_UNKNOWN_NOTES 	| 2101       	| The module received unknown counterfeit notes. 	|
| W_BA15DEV_CASSETTE_ALMOST_FULL 	| 2102       	| The cassette of the module is almost full (E56). |
| W_BA15DEV_NOTES_CLEAR_TO_CASSETTE 	| 2110       	| Notes are cleared to the cassette. |



<hr>


## **tBR15Dispense**
Structure of returning to output slot (change) (valid for br-15).
```c++
typedef struct {
    // number of notes to be dispensed from each cassette: suffix 0: drum 1; 1: drum 2; 2: drum 3......
    int aiDispenseNumber[8]; 

    // number of notes actually dispensed from each cassette: suffix 0: drum 1; 1: drum 2; 2: drum 3......
    int aiOutNumber[8]; 

    // number of invalid notes recovered during note dispensing in each cassette char acHopper[8]; // slot number of cassette
    int aiRejectNumber[8]; 
}tBR15Dispense; 
```

<hr>

## **tCashUnitInfo**
```c++
typedef struct
{
    // identification information, read from the configuration file of the module, e.g.: GU1
    char acCashUnitInfo[32];
    // cassette ID, read from the configuration file of the module, e.g.: 00005 
    char acCashUnitID[32];
    // currency, e.g.: CNY 
    char acCurrencyID[4]; 
    // cassette type: 1: recycling; 2: deposit-only, 3: replenish, 4: deposit and retract
    int iType; 
    // denomination, 0: any; *: specific denomination
    int iValue; 
    // current number of notes
    int iCount; 
    // number of unaccounted notes: valid for recycling drum
    int iNoAccount; 
    // maximum number
    int iMaximum; 
    // cassette status, 0: normal; 1: full; 2: to be full; 3: low; 4: empty; 5: unavailable; 6: inexistence; 7: invalid denomination
    int iStatus; 
}tCashUnitInfo;
```

Note: 

Sequence of the cassette units: 
```C++
    0: drum 1; 
    1: drum 2; 
    2: drum 3;
    3: drum 4; 
    4: replenishing cassette; 
    5: cassette
```

The maximum number of notes for drum 1 and drum of BR15 is 50, that of drum 3 and drum 4 is 100, ID information of drums are set based on the configuration file, and ID of the replenishing cassette and cassette is read from the module.

The number of notes for BR15 can be set to change, and the number of unaccounted notes can not be set.

<hr>

## **tCIMCapabilities**
Structure to return device capacity set.

```C++

typedef struct
{
    WORD wCIMMode; //1: support deposit only; 2: support retract only; 3: support deposit and withdrawal WORD wMaxCashInItem; // maximum number of accounted notes
    BOOL bRefill; // support automatic feeding: 1: support; 2: unsupported
    BOOL bInterStacker; // stacker: 1: yes 2: no
    BOOL bInterBuffer; // internal note buffer: 1: yes; 2: no
    BOOL bItemTakenSensor; // support Taken Sensor: 1: yes; 2: no
    CHAR cDevType[32]; // model, e.g.: GRG-BR15
    CHAR cDevVersion[128]; // hardware version
    CHAR cDLLVersion[32]; // driver libration version
    CHAR cSerialNumber[32]; // serial number, null for no hardware CHAR cProductDate[32]; // product date, null for no hardware
}tCIMCapabilities;
```


<hr>


## **tCIMCashInfo**
Structure to return the note informationt.

```c++
typedef struct
{
    char acCurrency[4]; // currency, e.g.: CNY
    int iCount; // serial number of count, enter serial number of notes in stacker (valid for br-15) long lDenomination; // denomination value
    char cSerial; // note version, e.g.: A
    int iDirection; // note direction: 0: drum 1 ... 4: stacker; 5: cassette; 6: BV(buffer); 7: output slot int iWay; // note direction: 0: unknown 1.. 2.. 3.. 4..
}tCIMCashInfo;

```

Note: When unrecognized notes are received, only iCount returns information, and other interface returns null information.


<hr>

## **tCIMOCRInfo**
Structure to return the information about the OCR image data.

```c

typedef struct
{
    char acORCInfo[40];  // OCR character string, i.e. serial number of a note
    PBYTE byImageBuffer; // address of the bmp image data, with the data space being applied and released by the dynamic library
    UINT uiBuferrLen;    // length of the bmp image data
}tCIMOCRInfo;

```

<hr>

## **tCIMStatusInfo**
Structure to return device status

```c

typedef struct
{
    WORD wDevice; (reserved)// current status of the module
    WORD wCashIn; // status of input slot: 1: there are notes; 2: no notes; 0: unknown
    WORD wCashOut; //status of output slot: 1: there are notes; 2: no notes; 0: unknown
    WORD wBuffer; //status of buffer (BV): 1: there are notes; 2: no notes; 0: unknown
    WORD wEnableMode; // receiving mode: 1: (ready to) receive is enabled; 2:(ready to) receive is closed; 0: unknown WORD wTransport; // transport status: 1: there are notes; 2: no notes; 0: unknown
    WORD wStacker; // stacker status: 1: there are notes; 2: no notes; 0: unknown
    int iStackerCount; // number of notes in stacker
}tCIMStatusInfo;

```

<hr>

## **tCIMCashProperty**
Structure of denomination information.

```c
typedef struct
{
        long lDenomCode; // serial number (reserved) 
        char acCurrency[4]; // currency, e.g.: CNY 
        long lDenomination; // denomination value 
        char cSerial; // currency version, e.g.: A
}tCIMCashProperty;

```

<hr>

## **[tDevReturn]*(#)
    Structure to return the device status when the command is correctly executed or return the error code when the command is incorrectly executed


```c++

typedef struct
{
        int iLogicCode; // logic error code
        int iPhyCode; // physical error code
        int iHandle; // handling methods: 0-do nothing; 1-initialize; 2-resend the command
        int iType; // type of error:: 0-warning, 1-serious
        char acDevReturn[128]; // returned information from the 1fzåazmzE   Q44hardware
    char acReserve[128]; // reserved information. It records the information about the original errors (errors only) actually returned from the hardware, and does not need to be converted.
}[tDevReturn]; //member descriptio(#)

```

<hr/>

## **CIM_CancelPrepare**
Cancel the module to enter the status of receiving notes.

```c
int Cim_CancelPrepare([tDevReturn]*p_psStatus)(#)
```

**Parameter description**

```p_psStatus```

 - [out] Returned status structure
 - For detailed information, see the structure [[tDevReturn]](#[tDevReturn](#)

**Returned Value**  
See Returned value of interface function

**Precautions**  
When the command is successfully executed, the green light will be off.

**Related Interfaces**  
[Cim_PrepareDeposit](#tDevReturn)


## **CIM_CashCount** (valid for br-15)
Count the notes in the module (drum) into the cassette.

```c++
int Cim_CashCount(PBYTE p_byCountNumber, tCashUnitInfo *p_psCashUnitInfo,tDevReturn* p_psStatus);
```
**Parameter description**

```p_byCountNumber```
 - [in/Out] 4-byte array, corresponding to number of notes to be counted by drum 1, drum 2, drum 3 and drum 4, and return the actual number of notes after actual counting

```p_psCashUnitInfo```
 - [out] Return to the cassette structure, and display the information about number of notes in each drum after counting
 - For detailed information, see the structure [tCashUnitInfo](#tCashUnitInfo)

```p_psStatus```
 - [out] Returned status structure
 - For detailed information, see the structure [tDevReturn](#tDevReturn)

 **Returned Value**  
See Returned value of interface function

**Precautions**  
If the number of notes to be counted is more than the number of notes in the drum, return a warning code.

 **Example**  
None

**Related Interfaces**  
[Cim_Replenish](#Cim_Replenish)


## **Cim_CashInTo**
Press notes in stacker or cassettet.

```c

int Cim_CashInTo(char p_pcPosition, tDevReturn* p_psStatus);

```

**Parameter description**
```p_pcPosition```
 - [in] 0x31 : to stacker; 0x32 : to cassette

```p_psStatus```
 - [out] Returned status structure
 - For detailed information, see the structure [tDevReturn](#tDevReturn)

  **Returned Value**  
See Returned value of interface function

**Precautions**  
Corresponding feeding operation is not available for the device without stacker.

**Related Interfaces**
[Cim_CashOutFrom](#Cim_CashOutFrom)


## **Cim_CashOutFrom**
Return notes from stacker or buffer.

```c++
int Cim_CashOutFrom(char p_pcPosition, tDevReturn* p_psStatus);
```

**Parameter description**
```p_pcPosition```
 - [in] 0x31 : from buffer (BV); 0x32 : from stacker
 - For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned Value**  
See Returned value of interface function

**Precautions**  
Corresponding feeding operation is not available for the device without stacker.

**Related Interfaces**
[Cim_CashInTo](#Cim_CashInTo)

## **Cim_CloseComm**
Close the open serial port.

```c
int Cim_CloseComm();
```

**Parameter description**   
None

**Returned Value**   
None

**Precautions**  
It is available only after calling Cim_SetCommPara after calling the interface.

**Example**  
None

**Related interface**   
[Cim_SetCommPara](#Cim_SetCommPara)


## **Cim_Dispense** (Valid for br-15)
Dispense notes from specified drum.

```c++
int Cim_Dispense(tBR15Dispense* p_psBR15Dispense, tDevReturn* p_psStatus);
```

**Parameter description**   
```p_psBR15Dispense```
- [in/out] Returned status structure
- For detailed information, see the structure [tBR15Dispense](#tBR15Dispense)

```p_psStatus```
 - [out] Returned status structure
 - For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned Value**   
See Returned value of interface function

**Precautions**  
1. The number of notes to be dispensed from BR15 is 20.
2. Return a warning when notes in the BR15 are insufficient, and check to see whether the number of dispensed
notes is consistent with the number of actually dispensed notes when the command is executed.

**Example**  
None

**Related interface**   
None

## **Cim_GetCapalities**
Get device capability set.

```c++
int Cim_GetCapalities(tCIMCapabilities* p_psCapablitiesInfo, tDevReturn* p_psStatus);
```

**Parameter description**   
```p_psCapablitiesInfo```
- [out] Returned status structure
- For detailed information, see the structure [tCIMCapabilities](#tCIMCapabilities)   

```p_psStatus```
 - [out] Returned status structure
 - For detailed information, see the structure [tDevReturn](#tDevReturn)

 **Returned Value**   
See Returned value of interface function


## **Cim_GetCashInfo**
Get received note information.

```c
int Cim_GetCashInfo(tCIMCashInfo *p_psCashInfo, int& p_iCashNumber, tDevReturn* p_psStatus );
```

**Parameter description**  
```p_psCashInfo```
- [out] Return note information structure.
- For detailed information, see the structure [tCIMCashInfo](#tCIMCashInfo)

```p_iCashNumber```
- [out] Return number of validated notes, constant 1 for banknote recycling module (when notes enter).

```p_psStatus```
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn)

Precautions
1. When receiving validated notes, the note will automatically return to the output slot, 1 for `p_iCashNumber`, and 0 (empty) for other information.
2. Notes stay in the buffer when `BA15` and `BIM2020` receive invalidated notes.
3. [tCIMCashInfo](#tCIMCashInfo) of BA-08 needs defining structure array with length up to 30.


## **Cim_GetCashOCRInfo**
Get the OCR image data of notes.

```c
int Cim_GetCashOCRInfo(WORD p_wCashNum , tCIMOCRInfo *p_psOCRInfo, tDevReturn* p_psStatus );
```

**Parameter description**

`p_wCashNum`  
- [in] Serial number for a note, get through iCount in CIM_GetCashInfo

`p_psOCRInfo`
- [out] Returned status structure
- For detailed information, see the structure [tCIMOCRInfo](#tCIMOCRInfo)

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned value**   
See Returned value of interface function

**Precautions**   
Confirm the relevant hardware support before using the function of getting the OCR image.


## **Cim_GetCashUnitInfo**
Get attribute of each cassette (recycling drum) of the device.

```c
int Cim_GetCashUnitInfo( tCashUnitInfo *p_psCashUnitInfo, tDevReturn* p_psStatus);
```

**Parameter description**
`p_psCashUnitInfo`
- [out] Returned status structure
- For detailed information, see the structure [tCashUnitInfo](#tCashUnitInfo)

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn) 

**Returned value**  
See Returned value of interface function 

**Precautions**
1. The cassette attribute information of `BA15` and `BIM2020` is the array suffix 0 position.
2. `BR15` has 6-length arrays, and other module uses only one attribute length.
3. The recycling drum of `BR15` is fixed to be recycling type.

**Related interface**
[Cim_SetCashUnitInfo](#Cim_SetCashUnitInfo)


## **Cim_Init**
Initialize the banknote recycling module, and call the interface when the device has a fault.

```c
int Cim_Init(const int p_iAction , tDevReturn* p_psStatus);
```

**Parameter description**
`p_iAction`
- [in] Specify the note direction when initialization: 1: return 2: pressure cassette

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned value**
See Returned value of interface function

**Precautions**  
`BR15` will read corresponding denomination and other information from the configuration file and set corresponding cassette (recycling drum). If such cassette unit is not empty and the set denomination information changes, set the cassette in unavailable status.


## Cim_PrepareDeposit**
Prepare the banknote recycling module to enter the status of receiving notes.

**Parameter description**
`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned value**  
See Returned value of interface function

**Precautions**  
When the command is successfully executed, the light will turn green. 

**Related interface**  
[Cim_CancelPrepare](#Cim_CancelPrepare)


## **CIM_Replenish** (valid for br-15)
Replenish notes in the cassette to the corresponding recycling drum.

```c++
int Cim_Replenish(
    int p_iCashUnit ,
    const WORD p_wNumber ,
    int& p_iRetract ,
    tCashUnitInfo *p_psCashUnitInfo, 
    tDevReturn* p_psStatus
);

```


**Parameter description**
`p_iCashUnit`
- [in] Specify drum to receiving notes replenished from the cassette: 0: automatically replenish to corresponding four drums: 1: to drum 1; 2:......

`p_wNumber`
[in] Number of notes to be replenished

`p_iRetract`
[out] Number of notes retracted during replenishing

`p_psCashUnitInfo`
- [out] Return information about number of notes in each drum For detailed information, see the structure [tCashUnitInfo](#tCashUnitInfo)

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn) 

**Returned value**
See Returned value of interface function Precautions
1. If notes are replenished to empty cassette, stop replenishing and return corresponding warning code.
2. The maximum number is 100.
3. Replenishing stops when consecutive five notes are recovered during replenishing.


## **Cim_SetCashUnitInfo** (valid for br-15) 
Set attribute of cassette unit (drum) in the device.

```c++
int Cim_SetCashUnitInfo(tCashUnitInfo *p_psCashUnitInfo, tDevReturn* p_psStatus);
```

**Parameter description**  
`p_psCashUnitInfo`
- [in] Returned status structure
- For detailed information, see the structure [tCashUnitInfo](#tCashUnitInfo)

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn) 

**Returned value**  
See Returned value of interface function

**Precautions**  
If there are notes in the set drum and the set denomination is inconsistent with the previous setting, set the drum to be in unavailable status.

Set the denomination to 0 to shield a drum.

Set the drum to be in unavailable status if the set denomination is invalid.

Initialize the device after set the cassette attribute.


## **Cim_SetCommPara**
Based on the logical device name set by the user, read the parameters to be set for communication from the relevant configuration file or registry, set the parameters for communication of the device, and establish a connection for communication with the device.

```c++
int Cim_SetCommPara(tDevReturn* p_psStatus );
```

**Parameter description**  

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned value**  
See Returned value of interface function

**Precautions**  
This interface is compatible with the Commanger3.0 version. All the settings of communication parameters are obtained from the `GRGDTATM_CommCfg.ini` file. See comments in the `GRGDTATM_CommCfg.ini` file for specific configuration methods.

```c++
[BILLDEPOSITDEV] //
COMMTYPE =1 //1:RS232 communcation, generally unchanged 
ComID =3 // corresponding port number in computer
ComBaud =9600 // baud rate, generally unchanged ComBoardPort =
ComBoardPortBaud =
DevCommLogID =2100
DevTraceLogID =2101
IniCfgFileName =BillDepositDevCfg.ini

```

**Related interface**  
[Cim_CloseComm](#Cim_CloseComm)


## **Cim_GetRecogniseAbility**
Get the denomination that cannot be recognized by the module currently, and then distinguish all denominations that can be recognized by the module. “The denomination that can be recognized by the module currently” is certainly included in “all denominations that can be recognized by the module”.

```c++
int CIM_GetRecogniseAbility(tCIMCashProperty* p_tCashProperty, int& p_iNum, tDevReturn* p_psStatus);
```

**Parameter description**  

`p_tCashProperty`
- [out] Returned denomination information structure. For detailed information, see the structure [tCIMCashProperty](#tCIMCashProperty).

`p_psStatus`
- [out] Returned status structure. For detailed information, see the structure [tDevReturn])(#tDevReturn).

`p_iNum`
- [out] Total number of recognized denominations.


**Returned value**  
See Returned value of interface function Precautions
1. Denominations that can be recognized by the module.
2. Structure [p_tCashProperty](#p_tCashProperty) needs certain assigned space, generally a structure array.

**Related interface**
1. [Cim_GetRecogniseNum](#Cim_GetRecogniseNum)
2. [Cim_GetAllRecogniseAbility](#Cim_GetAllRecogniseAbility)


## **Cim_SetRecogniseAbility**
Denominations that can be recognized by the module.

```c++
int Cim_SetRecogniseAbility(tCIMCashProperty* p_tCashProperty, int p_iNum, tDevReturn* p_psStatus );
```
**Parameter description**  

`p_tCashProperty`
- [in] Information structure of recognizable denominations to be set. 
- For detailed information, see the structure [tCIMCashProperty](#tCIMCashProperty).

`p_iNum`
- [in] Total number of recognizable denominations to be set

`p_psStatus`
- [out] Returned status structure，For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned value**  
See Returned value of interface function

**Precautions**  
1. If the set denomination is out of the supported denomination range of the module, the denomination will not take effect.
2. Structure [p_tCashProperty](#p_tCashProperty) needs certain assigned space, generally a structure array.

**Related interface**  
1. [Cim_GetRecogniseNum](#Cim_GetRecogniseNum)
2. [Cim_GetRecogniseAbility](#Cim_GetRecogniseAbility)


## **Cim_GetRecogniseNum**
Get number of denominations that can be recognized by the module.

```c
int Cim_GetRecogniseNum( int& p_iNum , tDevReturn* p_psStatus);
```

**Parameter description**  

`p_iNum`
- [out] Total number of recognized denominations.

`p_psStatus`
- [out] Returned status structure，For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned value**  
See Returned value of interface function

**Precautions**  
1. Distinguish number of denominations that can be recognized by the module. Example
None
Related interface
1. [Cim_GetRecogniseNum](#Cim_GetRecogniseNum)



## **Cim_GetAllRecogniseAbility**
Get the denomination that can be recognized by the module, and then distinguish all denominations that can be recognized by the module currently. “The denomination that can be recognized by the module currently” is certainly included in “all denominations that can be recognized by the module”.

```c++
int Cim_GetAllRecogniseAbility(
    tCIMCashProperty* p_tCashProperty, 
    int& p_iNum,
    tDevReturn* p_psStatus
);
```
**Parameter description**  

`p_tCashProperty`
- [out] Returned denomination information structure. 
- For detailed information, see the structure [tCIMCashProperty](#tCIMCashProperty).

`p_psStatus`
- [out] Returned status structure. 
- For detailed information, see the structure [tDevReturn](#tDevReturn)

`p_iNum`
- [out] Total number of recognized denominations.

**Returned value**  
See Returned value of interface function 

**Precautions**  
1. Distinguish denominations that can be recognized by the module currently.
2. Structure [p_tCashProperty](#p_tCashProperty) needs certain assigned space, generally a structure array.

**Related interface**
1. [Cim_GetAllRecogniseNum](#Cim_GetAllRecogniseNum)
2. [Cim_GetRecogniseAbility](#Cim_GetRecogniseAbility)


## **Cim_GetAllRecogniseNum**
Get number of denominations that can be recognized by the module.

**Parameter description**  

`p_iNum`
- [out] Total number of recognized denominations.

`p_psStatus`
- [out] Returned status structure. 
- For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned value**  
See Returned value of interface function

**Precautions**  
Distinguish number of denominations that can be recognized by the module currently. 

**Related interface**  
[Cim_GetAllRecogniseNum](#Cim_GetAllRecogniseNum)

## **Cim_GetStatus**
Get device status information.

**Parameter description**  

`p_psStatusInfo`
- [out] Returned status information structure. 
- For detailed information, see structure [tCIMStatusInfo](#tCIMStatusInfo).

`p_psStatus`
- [out] Returned status structure. 
- For detailed information, see structure [tDevReturn](#tDevReturn).

**Returned information**  
See Returned Information of Interface Parameter.

**Precautions**  
Bim2020 cannot get `wCashIn` status, and the `wCashOut` information can be got when returning notes, and
cannot be got when blocked artificially
  



## **Cim_ReconverCash**
Recover the notes at the output slot.

```c++
int Cim_ReconverCash(tCIMCashInfo *p_psCashInfo, int& p_iCashNumber, tDevReturn* p_psStatus );
```

**Parameter description**  

`p_psCashInfo`
- [out] Returned note information structure
- For detailed information, see the structure [tCIMCashInfo](#tCIMCashInfo)

`p_iCashNumber`
- [out] Return number of recovered notes, constant 1 for banknote recycling module.

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn) 

**Returned value**  
See Returned value of interface function

**Precautions**  
1. When receiving unrecognized notes,the note will automatically return to the output slot,then recover the note or take away the note.
2. Generally,after recovering notes,call interface [Cim_NormalSaveCashOCR](#Cim_NormalSaveCashOCR) to [OCRimage](#OCRimage) of the recovered note.


**Related interface**  
[CIM_NormalSaveCashOCR](#CIM_NormalSaveCashOCR)


## **CIM_NormalSaveCashOCR**
Get OCR information of note.

```c+
int Cim_NormalSaveCashOCR(tGRGNoteDetails *p_psNoteDetails, UINT &p_uiNumOfItems, tDevReturn* p_psStatus );
```

**Parameter description **  

`p_psNoteDetails`
- [In/Out] The upper layer shall assign space, and return the note information.

`p_uiNumOfItems`
- [In/Out] Length of note information (number of notes), return the number of the notes on which information is got.

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn)

**Returned value**  
See Returned value of interface function 

**Precautions**  
1. Confirm the relevant hardware support using the function of getting the OCR image.
2. Distinguish [CIM_GetNvNotesInfo](#CIM_GetNvNotesInfo). Call [CIM_GetNvNoteInfo](#CIM_GetNvNoteInfo) generally after calling the interface of getting note information [CIM_GetCashInfo](#CIM_GetCashInfo), call [CIM_NormalSaveCashOCR](#CIM_NormalSaveCashOCR) at any time, and get the OCR information of the note currently inserted.

**Related interface**  
[CIM_GetNvNotesInfo](#CIM_GetNvNotesInfo)

## **CIM_NormalSaveCashOCR**
**Get note information.**  
Get the note information validated by NV, including note prefix image, record database and written image.

```c++
int iGetNvNotesInfo( tGRGNoteDetails *p_psNoteDetails, UINT &p_uiNumOfItems,
BYTE p_byNoteType, tDevReturn* p_psStatus );
```

**Parameter description**   
`p_psNoteDetails`
- [In/Out] The upper layer shall assign space, and return the note information.

`p_uiNumOfItems`
- [In/Out] Length of note information (number of notes), return the number of the notes on which information is got.

`p_byNoteType`
- [In] Get note type:  
     `0x01:` genuine note;  
     `0x02:` non-genuine note;   
     `0x03:` all notes; these can be combined.   
     `0x04:` blacklist note, reserved.  

`p_psStatus`
- [out] Returned status structure
- For detailed information, see the structure [tDevReturn](#tDevReturn) 

**Returned value**  
See Returned value of interface function 

**Precautions**  
1. This interface is called after withdrawal; for the module with NV of the OCR function, this interface must be called after withdrawal to obtain the OCR information; to call an action related command such as “dispense note”, the lower layer will clear the relevant information in NV.
2. The parameter p_psCashInDetails is preferably defined as a 100-dimensional array and must be initialized.
3. This interface will record the note information in the database and generate the prefix image file.To keep the OCR function stable and reliable, the upper layer needs to call the iDeleteSerialRecords to delete the note information record interface and regularly clear the database and note prefix image. Please refer to the instructions on how to delete note information record interface.
4. The NV of the cash dispenser does not have the note recognition function and only supports image collection and transmission and prefix identification, therefore, the recognition results do not include the note denomination. When the driver writes the database, the denomination is set to be 0.
5. Now NV only supports recognition of RMB notes of 100 yuan denomination.
  
**Example**  

```c++

// Definition of device class
OGRGCDMDev g_oDev;

int l_iResult = 0;
DWORD l_dwTime = 0;
tDevReturn l_asDevReturn[8] = {0};

// set communication parameter. g_oDev.iSetCommPara(l_asDevReturn);
if(SUCCESS  != l_iResult){
    return
}

// Initialize g_oDev.iInit(l_asDevReturn);
if (SUCCESS != l_iResult){
    return;
}

tGRGTransInfoParas l_sTransInfo;
memset(&l_sTransInfo, 0, sizeof(l_sTransInfo));
SYSTEMTIME l_sTime;
GetLocalTime(&l_sTime);

sprintf(l_sTransInfo.acJournalNo, "%04d%02d%02d_%02d%02d%02d_%03d", 
    l_sTime.wYear,
    l_sTime.wMonth,
    l_sTime.wDay,
    l_sTime.wHour,
    l_sTime.wMinute,
    l_sTime.wSecond,
    l_sTime.wMilliseconds
);

l_sTransInfo.byTimes = 1;
g_oDev.iSetTransInfo(&l_sTransInfo, l_asDevReturn);

// Dispense
tDispense l_sDispense;
memset(
    &l_sDispense, 0, 
    sizeof(l_sDispense)); 
    l_sDispense.acHopper[0] = 1;
    l_sDispense.acHopper[1] = 2;
    l_sDispense.acHopper[2] = 3;
    l_sDispense.acHopper[3] = 4; m_sDispense.aiDispenseNumber[0] = 1;
    l_iResult = g_oDev.iDispense(&l_sDispense, l_asDevReturn
);

if (SUCCESS != l_iResult)
{
r   eturn;
}

// Get OCR recognition information
tGRGNoteDetails l_asDetails[100] = {0};
int l_uiCount = 100;
l_iResult = g_oDev.iGetNvNotesInfo(l_asDetails, l_uiCount, 1, l_asDevReturn);

```

**[DllExtFunc] Driver logging confirmation**

1. `[DevDriverLog]`: logging or not   
- 0: not logging
- 3: logging  

    If it is configured to logging, the DevDriverLog.dll files need to be copied to the same-level directory of the dynamic library.

2. `[RemainDays]`: period of validity of recorded log It defaults to be 7 days.

3. [TraceLevel] Logging level
- `0`: Log records dynamic library version information
- `1`: driver interface returns information recorded at error
- `2`: driver interface returns information recorded at warning 
- `3`: driver interface returns information recorded at warning 
- `4`: record all information

`BillDepositDevCfg.ini` configuration file is as follows:

```c++

; Select currently used device: 
    1:BA-15 
    2:Bim2020 
    3:Mei 
    4:BA08 
    5:BR15 [BillDepositType]

BillDepositType = 5[OCR];
0: disable the OCR function; 
1: enable the OCR function 
OCREnable = 0
DeviceGUID ={77F49320-16EF-11d2-AD51-006097B514DD} 
SendPipes =00
RevPipes =02
ExtendStr =VID:04b4,PID:8006; 

below is valid for BR15
[1]
CashUnitInfo = GU1 // customize name information CashUnitID = 00001 // customize ID information CurrencyID = CNY // currency
Value = 5 // donomination
Maximum = 50 // maximum number

[2]
CashUnitInfo = GU2 
CashUnitID = 00010 
CurrencyID = CNY 
Value = 10 
Maximum = 50

[3]
CashUnitInfo = GU3 
CashUnitID = 00050 
CurrencyID = CNY
Value = 50 
Maximum = 100

[4]
CashUnitInfo = GU4 
CashUnitID = 00020 
CurrencyID = CNY 
Value = 20 
Maximum = 100

[5]
CashUnitInfo = REP 
CashUnitID = 88888 
CurrencyID = CNY 
Value = 0
Maximum = 500

[6]
CashUnitInfo = CAS 
CashUnitID = 99999 
CurrencyID = CNY 
Value = 0
Maximum = 800

```
