# HandleLeak

先看一段，这个有没有问题？
```cpp
	SHELLEXECUTEINFO ShExecInfo = {0};
	ShExecInfo.cbSize = sizeof(SHELLEXECUTEINFO);
	ShExecInfo.fMask = SEE_MASK_NOCLOSEPROCESS;
	ShExecInfo.hwnd = NULL;
	ShExecInfo.lpVerb = _T("open");
	ShExecInfo.lpFile = _T("cmd");
	ShExecInfo.lpParameters = _T("/c echo 111");
	ShExecInfo.lpDirectory = NULL;
	ShExecInfo.nShow = SW_HIDE;
	ShExecInfo.hInstApp = NULL;

	ShellExecuteEx(&ShExecInfo);

	WaitForSingleObject(ShExecInfo.hProcess,INFINITE); 
```
答案是有问题的，问题在于设置了`SEE_MASK_NOCLOSEPROCESS`
```cpp
typedef struct _SHELLEXECUTEINFOW
{
    DWORD cbSize;               // in, required, sizeof of this structure
    ULONG fMask;                // in, SEE_MASK_XXX values
    HWND hwnd;                  // in, optional
    LPCWSTR  lpVerb;            // in, optional when unspecified the default verb is choosen
    LPCWSTR  lpFile;            // in, either this value or lpIDList must be specified
    LPCWSTR  lpParameters;      // in, optional
    LPCWSTR  lpDirectory;       // in, optional
    int nShow;                  // in, required
    HINSTANCE hInstApp;         // out when SEE_MASK_NOCLOSEPROCESS is specified
    void *lpIDList;             // in, valid when SEE_MASK_IDLIST is specified, PCIDLIST_ABSOLUTE, for use with SEE_MASK_IDLIST & SEE_MASK_INVOKEIDLIST
    LPCWSTR  lpClass;           // in, valid when SEE_MASK_CLASSNAME is specified
    HKEY hkeyClass;             // in, valid when SEE_MASK_CLASSKEY is specified
    DWORD dwHotKey;             // in, valid when SEE_MASK_HOTKEY is specified
    union                       
    {                           
        HANDLE hIcon;           // not used
#if (NTDDI_VERSION >= NTDDI_WIN2K)
        HANDLE hMonitor;        // in, valid when SEE_MASK_HMONITOR specified
#endif // (NTDDI_VERSION >= NTDDI_WIN2K)
    } DUMMYUNIONNAME;           
    HANDLE hProcess;            // out, valid when SEE_MASK_NOCLOSEPROCESS specified
} SHELLEXECUTEINFOW, *LPSHELLEXECUTEINFOW;
```

注意看`HANDLE hProcess;// out, valid when SEE_MASK_NOCLOSEPROCESS specified`

也就是说，如果指定了`SEE_MASK_NOCLOSEPROCESS`，`hProcess`就是返回的句柄。如果不关闭，就会造成句柄泄漏。

测试截图
