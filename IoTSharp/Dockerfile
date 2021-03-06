#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:5.0-buster-slim AS base
MAINTAINER Yanhong Ma 2020
RUN echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free" > /etc/apt/sources.list && \
	echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free" >> /etc/apt/sources.list && \
	echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free" >> /etc/apt/sources.list && \
	echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free" >> /etc/apt/sources.list && \
	echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security jessie/updates main contrib non-free" >> /etc/apt/sources.list && \
    apt-get  -y   -q update   && apt-get install  -y   -q  apt-utils libgdiplus libc6-dev lsof net-tools wget sqlite3 python3  bashtop   iputils-ping inetutils-tools  libssl-dev  libssl1.0.0 curl  && \
	apt-get autoremove -y &&  apt-get clean  &&  apt-get autoclean && rm  /var/cache/apt/* -rf &&  \
	ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN curl -o TDengine-client.tar.gz "https://www.taosdata.com/download/download-gettingStarted.php?pkg=tdengine_linux&pkgName=TDengine-client-2.0.12.0-Linux-x64.tar.gz" && \
   tar -xzvf TDengine-client.tar.gz && rm TDengine-client.tar.gz -f  && cd  $(ls TDengine-client*  -d) && ./install_client.sh && \
    rm $(pwd) -rf
WORKDIR /app
EXPOSE 80  
EXPOSE 1883 
EXPOSE 8883 
EXPOSE 502 
EXPOSE 5683
EXPOSE 5684 
EXPOSE 8080


FROM mcr.microsoft.com/dotnet/sdk:5.0-buster-slim AS build
RUN echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free" > /etc/apt/sources.list && \
	echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free" >> /etc/apt/sources.list && \
	echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free" >> /etc/apt/sources.list && \
	echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free" >> /etc/apt/sources.list && \
	echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security jessie/updates main contrib non-free" >> /etc/apt/sources.list && \
    apt-get  -y   -q update   && apt-get install  -y   -q  git  libssl-dev  libcurl3 libssl1.0.0   && \
	apt-get autoremove -y &&  apt-get clean  &&  apt-get autoclean && rm  /var/cache/apt/* -rf && \
	ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
WORKDIR /src
COPY ["IoTSharp/IoTSharp.csproj", "IoTSharp/"]
RUN dotnet restore  "IoTSharp/IoTSharp.csproj"  /clp:PerformanceSummary
COPY . .
WORKDIR "/src/IoTSharp"
RUN dotnet build "IoTSharp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "IoTSharp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "IoTSharp.dll"]
