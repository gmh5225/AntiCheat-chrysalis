# chrysalis
A small client-side user-mode anti-cheat.

# DETECTION WISHLIST #
----------------------

- [ ] Detect IAT hooks by taking a snapshot of the IAT at process start  
	* Maybe include some other parts of the PEB too?
	* Think about how this might be exploitable.. something feels off about it
- [ ] Detect manual mapped or unlinked modules by walking pages and checking if it falls outside the memory range of whitelisted modules
	* Look into how cheat engine / other memory scanners efficiently walk the pages
	* This is to detect modules that aren't linked in the PEB
	* The 'whitelisted' part of this may be tricky if a game supports mods
- [ ] Detect inline hooks & other .text section modifications by walking pages and checking if a page was previously copy-on-write
	* "If a shared copy-on-write page is modified, it becomes private to the process that modified the page. However, the VirtualQuery function will continue to report such pages as MEM_MAPPED (for data views) or MEM_IMAGE (for executable image views) rather than MEM_PRIVATE. To detect whether copy-on-write has occurred for a specific page, either access the page or lock it using the VirtualLock function to make sure the page is resident in memory, then use the QueryWorkingSetEx function to check the Shared bit in the extended working set information for the page. If the Shared bit is clear, the page is private."
- [ ] Detect suspicious external processes by enumerating all handles on system and looking for those that match our game process, and then look for additional criteria
	* Figure out some criteria to then look for in each process to determine if it's suspicious or not
	* Also be aware process hollowing exists, so a seemingly innocent process that rightfully has a handle to our game may in fact contain malicious code
- [ ] Detect external overlays by scanning all windows on the system and look for transparent, overlaid windows that match our game window's position and size
- [ ] Detect injection by hooking BaseThreadInitThunk in our game process to detect when a potentially malicious thread is created (check if its entrypoint is outside any official module's memory range or if its entrypoint is LoadLibrary w/ argument pointing to a suspicious dll file)
- [ ] Detect common vtable hooks by hashing d3d vtables
- [ ] Detect common render hooks by hashing d3d and openGL functions
	* Honestly, the copy-on-write detection should be good enough for this, but redundancy is good too
- [ ] Detect process hollowing by comparing key system processes to their exe file on disk
	* Can also do copy-on-write detection in these external processes, too. Probably very expensive.
- [ ] Look into global winapi hooks
	* Imagine if we could block OpenProcess calls to our process from ring3, or strip the handle access rights at least
- [ ] Look into how software breakpoint / hardware breakpoint hooks are done and formulate a detection mechanism
- [ ] Look into how window hijacking is done
	* Something about swapping gdi context in ring0 i think?
- [ ] Look into ETW information
- [ ] Look into USN journal records
- [ ] Look into possible detection mechanisms for thread hijacking (only threads associated with our own process)

# SECURITY WISHLIST #
---------------------

- [ ] Erase PE header from all anticheat modules
	* Manually map all modules too, never link with PEB.
- [ ] Custom GetProcAddress using hash as dll and function
- [ ] Use syscalls when possible
- [ ] Use encrypted structures for all global data
- [ ] Calls to our own functions should be encrypted too
- [ ] Look into hiding the presence of our own threads

# DESIGN WISHLIST #
-------------------

- [ ] Service process that communicates with our detection modules in the game process
	* Heartbeat and detection notifications, and streaming new modules in?
- [ ] Game launcher that registers the game process with the service process?
	* Or the service process can just detect when the game starts, but race conditions may exist w/ any cheat doing the same thing?
- [ ] Manually map each detection module into the game process
	* Should this be done by the service process or by some loader module in the game process?
- [ ] Stream each module over network and then into memory
- [ ] Figure out how to secure the service process too...
	* Really depends on the design here. What exactly do we want the service process to be responsible for?



# HOOK CANDIDATES #
-------------------

[game] hk_BaseThreadInitThunk (Kernel32ThreadInitThunkFunction - ntdll.dll)  
[game] hk_D3DXCreateFontA (EAT Hook)  
[game] hk_D3DXCreateFontIndirectA (EAT Hook)  
[game] hk_D3DXCreateSprite (EAT Hook)  
[game] hk_D3DXCreateTextureFromFileInMemory (EAT Hook)  
[game] hk_D3DXCreateTextureFromFileInMemoryEx (EAT Hook)  
[game] hk_D3DXLoadSurfaceFromMemory (EAT Hook)  
[game] hk_Dllmain_mono_dll (Inline Hook)  
[game?] hk_LoadAppInitDlls (Inline Hook)  
[game? or suspected injector?] hk_LoadLibraryExW_user32 (IAT Hook - user32.dll)  
[game? or suspected injector?] hk_LoadLibraryExW_ws2_32 (IAT Hook - ws2_32.dll)  
[?] hk_LockResource_kernel32 (IAT Hook - kernel32.dll)  
[suspected cheat process] hk_NtCreateFile_kernelbase (IAT Hook - kernelbase.dll)  
[suspected cheat process] hk_NtDeviceIoControlFile_mswsock (IAT Hook - mswsock.dll)  
[suspected cheat process] hk_NtOpenFile_kernelbase (IAT Hook - kernelbase.dll)  
[game & suspected cheat process] hk_NtProtectVirtualMemory_kernelbase (IAT Hook - kernelbase.dll)  
[?] hk_NtQueryDirectoryFile_kernelbase (IAT Hook - kernelbase.dll)  
[game] hk_NtUserGetAsyncKeyState_user32 (IAT Hook - user32.dll)  
[suspected cheat process & game?] hk_NtUserSendInput_user32 (IAT Hook - user32.dll)  
[?] hk_QueryPerformanceCounter (IAT Hook - game.exe)  
[?] hk_RtlExitUserProcess_kernel32 (IAT Hook - kernel32.dll)  
[game & suspected cheat process?] hk_VirtualAlloc_iat_kernel32 (IAT Hook - kernel32.dll)  