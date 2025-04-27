# github-actions-csharp
깃허브액션 C# 버전

## 깃허브액션 C#(.NET) 작업

1. 기본 리포지토리 VisualStudio로 생성
2. 닷넷 프로젝트 생성 및 코딩
    - WPF 프로젝트로 생성
        - VS에서 생성하면 솔루션 폴더도 생성되므로 생성 후 폴더 정리할 것
    - 깃헙 리포지토리를 루트로 진행

3. Actions 탭 이동
    - .NET으로 검색 후 .NET Desktop의 Configure 클릭

    <img src="./image/ga0001.png" width="800">

4. 리포지토리 내 .github/workflows/dotnet-desktop.yml 생성확인 원본 내용

    ```yaml
    # This workflow uses actions that are not certified by GitHub.
    # 이 워크플로우는 Github 인증을 받지 않은 액션을 사용함
    # 이 워크플로우는 .NET Core 기반으로 개발된 WPF나 윈폼 데스크탑 앱을 빌드, 테스트, 서명, 패키징하는 과정 수행
    # https://docs.microsoft.com/en-us/dotnet/desktop-wpf/migration/convert-project-from-net-framework
    # 공식 문서 URL을 참고할 것
    # 
    # 워크플로우 설정
    #
    # 1. 환경 변수 설정
    # 워크플로우 실행 시 기본환경변수를 자동으로 설정. 
    # env 색션에서 프로젝트에 맞게 변수를 수정할 것
    # 2. 코드 서명(signing)
    # Windows Application Packaging Project 에서 서명용 인증서를 생성하거나,
    # 기존의 서명 인증서를 프로젝트에 추가
    # 이후 PowerShell을 이용, .pfx 파일을 Base64로 인코딩해야 함
    # $pfx_cert = Get-Content '.\SigningCertificate.pfx' -Encoding Byte
    # [System.Convert]::ToBase64String($pfx_cert) | Out-File 'SigningCertificate_Encoded.txt'
    #
    # SigningCertificate_Encoded.txt 파일을 열어서
    # GitHub 보안문자를 "Base64_Encoded_Pfx"로 설정해서 추가
    # 더 자세한 내용은 URL참조 https://github.com/microsoft/github-actions-for-desktop-apps#signing
    #
    # 데스크탑 애플리케이션을 위한 GitHub Action 워크플로우의 전체 CI/CD 샘플을 보고 싶다면
    # URL 참조 https://github.com/microsoft/github-actions-for-desktop-apps

    name: .NET Core Desktop

    on:
    push:
        branches: [ "main" ]
    pull_request:
        branches: [ "main" ]

    jobs:

    build:

        strategy:
        matrix:
            configuration: [Debug, Release]

        runs-on: windows-latest  # For a list of available runner types, refer to
                                # https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idruns-on

        env:
        Solution_Name: your-solution-name                         # Replace with your solution name, i.e. MyWpfApp.sln.
        Test_Project_Path: your-test-project-path                 # Replace with the path to your test project, i.e. MyWpfApp.Tests\MyWpfApp.Tests.csproj.
        Wap_Project_Directory: your-wap-project-directory-name    # Replace with the Wap project directory relative to the solution, i.e. MyWpfApp.Package.
        Wap_Project_Path: your-wap-project-path                   # Replace with the path to your Wap project, i.e. MyWpf.App.Package\MyWpfApp.Package.wapproj.

        steps:
        - name: Checkout
        uses: actions/checkout@v4
        with:
            fetch-depth: 0

        # Install the .NET Core workload
        - name: Install .NET Core
        uses: actions/setup-dotnet@v4
        with:
            dotnet-version: 8.0.x

        # Add  MSBuild to the PATH: https://github.com/microsoft/setup-msbuild
        - name: Setup MSBuild.exe
        uses: microsoft/setup-msbuild@v2

        # Execute all unit tests in the solution
        - name: Execute unit tests
        run: dotnet test

        # Restore the application to populate the obj folder with RuntimeIdentifiers
        - name: Restore the application
        run: msbuild $env:Solution_Name /t:Restore /p:Configuration=$env:Configuration
        env:
            Configuration: ${{ matrix.configuration }}

        # Decode the base 64 encoded pfx and save the Signing_Certificate
        - name: Decode the pfx
        run: |
            $pfx_cert_byte = [System.Convert]::FromBase64String("${{ secrets.Base64_Encoded_Pfx }}")
            $certificatePath = Join-Path -Path $env:Wap_Project_Directory -ChildPath GitHubActionsWorkflow.pfx
            [IO.File]::WriteAllBytes("$certificatePath", $pfx_cert_byte)

        # Create the app package by building and packaging the Windows Application Packaging project
        - name: Create the app package
        run: msbuild $env:Wap_Project_Path /p:Configuration=$env:Configuration /p:UapAppxPackageBuildMode=$env:Appx_Package_Build_Mode /p:AppxBundle=$env:Appx_Bundle /p:PackageCertificateKeyFile=GitHubActionsWorkflow.pfx /p:PackageCertificatePassword=${{ secrets.Pfx_Key }}
        env:
            Appx_Bundle: Always
            Appx_Bundle_Platforms: x86|x64
            Appx_Package_Build_Mode: StoreUpload
            Configuration: ${{ matrix.configuration }}

        # Remove the pfx
        - name: Remove the pfx
        run: Remove-Item -path $env:Wap_Project_Directory\GitHubActionsWorkflow.pfx

        # Upload the MSIX package: https://github.com/marketplace/actions/upload-a-build-artifact
        - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
            name: MSIX Package
            path: ${{ env.Wap_Project_Directory }}\AppPackages

    ```

5. 참조링크 내 소스로 작성
    - [참조링크](https://github.com/microsoft/github-actions-for-desktop-apps/blob/main/.github/workflows/ci.yml)

    