# systemd - System and Service Manager

<a href="https://in.waw.pl/systemd-github-state/systemd-systemd-issues.svg"><img align="right" src="https://in.waw.pl/systemd-github-state/systemd-systemd-issues-small.svg" alt="Count of open issues over time"></a>
<a href="https://in.waw.pl/systemd-github-state/systemd-systemd-pull-requests.svg"><img align="right" src="https://in.waw.pl/systemd-github-state/systemd-systemd-pull-requests-small.svg" alt="Count of open pull requests over time"></a>
[![Semaphore CI Build Status](https://semaphoreci.com/api/v1/projects/28a5a3ca-3c56-4078-8b5e-7ed6ef912e14/443470/shields_badge.svg)](https://semaphoreci.com/systemd/systemd)<br/>
[![Coverity Scan Status](https://scan.coverity.com/projects/350/badge.svg)](https://scan.coverity.com/projects/350)<br/>
[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/1369/badge)](https://bestpractices.coreinfrastructure.org/projects/1369)<br/>
[![Travis CI Build Status](https://travis-ci.org/systemd/systemd.svg?branch=master)](https://travis-ci.org/systemd/systemd)<br/>
[![Language Grade: C/C++](https://img.shields.io/lgtm/grade/cpp/g/systemd/systemd.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/systemd/systemd/context:cpp)<br/>
[![CentOS CI Build Status](https://ci.centos.org/buildStatus/icon?job=systemd-pr-build)](https://ci.centos.org/job/systemd-pr-build/)

## Details

General information about systemd can be found in the [systemd Wiki](https://www.freedesktop.org/wiki/Software/systemd).

Information about build requirements is provided in the [README file](README).

Consult our [NEWS file](NEWS) for information about what's new in the most recent systemd versions.

Please see the [Hacking guide](docs/HACKING.md) for information on how to hack on systemd and test your modifications.

Please see our [Contribution Guidelines](docs/CONTRIBUTING.md) for more information about filing GitHub Issues and posting GitHub Pull Requests.

When preparing patches for systemd, please follow our [Coding Style Guidelines](docs/CODING_STYLE.md).

If you are looking for support, please contact our [mailing list](https://lists.freedesktop.org/mailman/listinfo/systemd-devel) or join our [IRC channel](irc://irc.freenode.org/%23systemd).

Stable branches with backported patches are available in the [stable repo](https://github.com/systemd/systemd-stable).

The only modification of this fork is, that we claim to be macOS. The code has been copied from [0xbb](github.com/0xbb/apple_set_os.efi) and pasted.

Diff:


    diff --git a/src/boot/efi/boot.c b/src/boot/efi/boot.c

		index 5aae7e7c0..a5d4421f3 100644
	--- a/src/boot/efi/boot.c
	+++ b/src/boot/efi/boot.c
	@@ -2073,6 +2073,50 @@ static VOID config_write_entries_to_variable(Config *config) {
		 (void) efivar_set_raw(&loader_guid, L"LoaderEntries", buffer, (UINT8*) p - (UINT8*) buffer, FALSE);
	 }
	 
	+#define APPLE_SET_OS_VENDOR	"Apple Inc."
	+#define APPLE_SET_OS_VERSION	"Mac OS X 10.9"
	+
	+static EFI_GUID APPLE_SET_OS_GUID = { 0xc5c5da95, 0x7d5c, 0x45e6, { 0xb2, 0xf1, 0x3f, 0xd5, 0x2b, 0xb1, 0x00, 0x77 }}; 
	+
	+typedef struct efi_apple_set_os_interface {
	+	UINT64 version;
	+	EFI_STATUS (EFIAPI *set_os_version) (IN CHAR8 *version);
	+	EFI_STATUS (EFIAPI *set_os_vendor) (IN CHAR8 *vendor);
	+} efi_apple_set_os_interface;
	+
	+EFI_STATUS
	+efi_apple_set_os(EFI_HANDLE image, EFI_SYSTEM_TABLE *systemTable)
	+{
	+	SIMPLE_TEXT_OUTPUT_INTERFACE *conOut = systemTable->ConOut;
	+	conOut->OutputString(conOut, L"apple_set_os started\r\n");
	+
	+	efi_apple_set_os_interface *set_os = NULL;
	+
	+	EFI_STATUS status  = systemTable->BootServices->LocateProtocol(&APPLE_SET_OS_GUID, NULL, (VOID**) &set_os);
	+	if(EFI_ERROR(status) || set_os == NULL) {
	+		conOut->OutputString(conOut, L"Could not locate the apple set os protocol.\r\n");
	+		return status;
	+	}
	+
	+	if(set_os->version != 0){
	+		status = set_os->set_os_version((CHAR8 *) APPLE_SET_OS_VERSION);
	+		if(EFI_ERROR(status)){
	+			conOut->OutputString(conOut, L"Could not set version.\r\n");
	+			return status;
	+		}
	+		conOut->OutputString(conOut, L"Set os version to " APPLE_SET_OS_VERSION  ".\r\n");
	+	}
	+
	+	status = set_os->set_os_vendor((CHAR8 *) APPLE_SET_OS_VENDOR);
	+	if(EFI_ERROR(status)){
	+		conOut->OutputString(conOut, L"Could not set vendor.\r\n");
	+		return status;
	+	}
	+	conOut->OutputString(conOut, L"Set os vendor to " APPLE_SET_OS_VENDOR  ".\r\n");
	+
	+	return EFI_SUCCESS;
	+}
	+
	 EFI_STATUS efi_main(EFI_HANDLE image, EFI_SYSTEM_TABLE *sys_table) {
		 static const UINT64 loader_features =
			 (1ULL << 0) | /* I honour the LoaderConfigTimeout variable */
	@@ -2094,6 +2138,9 @@ EFI_STATUS efi_main(EFI_HANDLE image, EFI_SYSTEM_TABLE *sys_table) {
		 BOOLEAN menu = FALSE;
		 CHAR16 uuid[37];
	 
	+	efi_apple_set_os(image, sys_table);
	+
	+
		 InitializeLib(image, sys_table);
		 init_usec = time_usec();
		 efivar_set_time_usec(L"LoaderTimeInitUSec", init_usec);
