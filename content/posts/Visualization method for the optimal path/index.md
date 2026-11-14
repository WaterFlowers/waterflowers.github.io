---
title: "Visualization method for the optimal path"
date: 2026-06-05
draft: false
tags: ["Visualization", "optimal path", "c++"]
summary: "Visualization method for the optimal path"
---

## Visualization method for the optimal path
---

## 最优路径的可视化方法
optimal_route.geojson  

### 一、实现方法

#### 1、在 UE5 中读取 GeoJSON 文件：
用 C++ 代码解析 optimal_route.geojson，提取其中的 LineString 坐标序列。   
#### 2、将经纬度转换为 UE5 世界坐标：
用 CesiumGeoreference 的 TransformLongitudeLatitudeHeightToUnreal 函数，将每个经纬度点（高度设为地形上方一定值，如50米）转换为引擎内的三维坐标。
#### 3、创建一条可视的线：
##### 方法一（推荐）：
创建一个 Actor，挂载 USplineComponent，将转换后的坐标依次设置为样条线的点。然后可以用一个半透明的发光材质渲染这条线，使其在三维地形上清晰可见。
##### 方法二：
使用cesium的 CesiumGlobeAnchor 或 CesiumCartographicPolygon 组件，但后者更适合面。样条线更灵活。

### 二、效果展示：

#### 1、对比历史仓储点
##### （1）数据准备：（史料记载和考古数据等）
唐代真实粮仓的坐标（如长安太仓、东渭桥仓、洛阳含嘉仓等）。
##### （2）做法：
在 Cesium 上，用 红色/金色光柱 标记这些历史仓储点。   
模型计算出的 TOP候选点（从仓储选址适宜性评价中得到）与历史仓储点叠加显示。如果模型推荐点与真实遗址空间位置接近，就直观验证了模型的有效性。   
#### 2、在地图上交互选址，自动计算成本
进阶功能：
##### （1）数据准备：
在 Python 离线分析时，不仅生成最终适宜性热力图，还生成一个 查询表（CSV），记录每个像素（或采样点）的经纬度、坡度、距河距离、坡度成本、水系成本、综合成本等字段。
##### （2）UE5 交互：
用户在数据洞察模式下，点击 Cesium 地形上的任意位置。   
查询逻辑：C++ 代码获取点击处的经纬度，在 CSV 表中查找最近的记录，读取该点的坡度、距河距离和成本值。   
显示结果：弹出一个 UMG 面板，显示：    
坡度：X.X°   
距最近河流：XXX 米   
综合建仓成本：XX.X   
适宜性等级：高/中/低   
决策支持：用户可以看到，如果在这里建仓，成本是多少，是否合适。实现“系统自动计算选择点的成本数据”。   
为避免实时读取大CSV， UE5 启动时一次性将 CSV 加载到内存（使用 TMap 或空间索引），点击时快速查找。   
#### 3、价值体现
（1）路径显示：直观证明成本距离分析算法有效，并展示了 C++/UE5 与 Python 分析的集成能力。    
（2）历史对比：体现了跨学科研究的严谨性——用数据科学方法验证历史决策。    
（3）交互选址：将静态分析升级为 探索式决策支持系统，实现“从分析到决策”的闭环。    

### 三、关键代码
```cpp
.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Components/SplineComponent.h"
#include "OptimalRouteActor.generated.h"

UCLASS()
class YOURPROJECT_API AOptimalRouteActor : public AActor
{
    GENERATED_BODY()
public:
AOptimalRouteActor();
 // 蓝图调用：解析 GeoJSON 文件，返回经纬度点数组（X=经度，Y=纬度）
    UFUNCTION(BlueprintCallable, Category = "Route")
    TArray<FVector2D> ParseGeoJSON(const FString& FilePath);
 // 蓝图调用：设置路径点（世界坐标），并生成样条线
    UFUNCTION(BlueprintCallable, Category = "Route")
    void SetRoutePoints(const TArray<FVector>& WorldPoints);
protected:
    virtual void BeginPlay() override;

   UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Route")
    USplineComponent* SplineComponent;
    UPROPERTY()
    TArray<class USplineMeshComponent*> SplineMeshes;
    void BuildSplineMeshes();
};
```
```cpp
.cpp
#include "OptimalRouteActor.h"
#include "Components/SplineComponent.h"
#include "Components/SplineMeshComponent.h"
#include "Engine/StaticMesh.h"
#include "Misc/FileHelper.h"
#include "Serialization/JsonReader.h"
#include "Serialization/JsonSerializer.h"

AOptimalRouteActor::AOptimalRouteActor()
{
    PrimaryActorTick.bCanEverTick = false;
//创建样条线
    SplineComponent = CreateDefaultSubobject<USplineComponent>(TEXT("SplineComponent"));
    SetRootComponent(SplineComponent);
}

// 解析 GeoJSON，返回经纬度数组
TArray<FVector2D> AOptimalRouteActor::ParseGeoJSON(const FString& FilePath)
{
    TArray<FVector2D> Result;

    // 读取文件内容
    FString JSONString;
    if (!FFileHelper::LoadFileToString(JSONString, *FilePath))
{
UE_LOG(LogTemp, Warning, TEXT("无法读取 GeoJSON 文件: %s"), *FilePath);
        return Result;
    }
    // 解析 JSON
TSharedPtr<FJsonObject>JsonObject;
TSharedRef<TJsonReader<>>Reader=TJsonReaderFactory<>::Create(JSONString);
    if (!FJsonSerializer::Deserialize(Reader, JsonObject) || !JsonObject.IsValid())
    {
        UE_LOG(LogTemp, Warning, TEXT("GeoJSON 解析失败"));
        return Result;
    }

    // 获取 features 数组
    const TArray<TSharedPtr<FJsonValue>>* FeaturesArray;
    if (!JsonObject->TryGetArrayField(TEXT("features"), FeaturesArray) || FeaturesArray->Num() == 0)
    {
        UE_LOG(LogTemp, Warning, TEXT("GeoJSON 中没有 features 数组"));
        return Result;
    }
    // 取第一个 feature
    TSharedPtr<FJsonObject> FirstFeature = (*FeaturesArray)[0]->AsObject();
    const TSharedPtr<FJsonObject>* GeometryObject = nullptr;
    if (!FirstFeature->TryGetObjectField(TEXT("geometry"), GeometryObject))
    {
        UE_LOG(LogTemp, Warning, TEXT("没有 geometry 字段"));
        return Result;
    }
    // 获取 coordinates 数组
    const TArray<TSharedPtr<FJsonValue>>* CoordinatesArray;
    if (!(*GeometryObject)->TryGetArrayField(TEXT("coordinates"), CoordinatesArray))
    {
        UE_LOG(LogTemp, Warning, TEXT("没有 coordinates 数组"));
        return Result;
    }

    // 遍历坐标
    for (int i = 0; i < CoordinatesArray->Num(); i++)
    {
        const TArray<TSharedPtr<FJsonValue>>* PointArray;
        if (!(*CoordinatesArray)[i]->TryGetArray(PointArray) || PointArray->Num() < 2)
            continue;

        double Longitude = (*PointArray)[0]->AsNumber();
        double Latitude = (*PointArray)[1]->AsNumber();

        // 存入经纬度，X=经度，Y=纬度
        Result.Add(FVector2D(Longitude, Latitude));
    }

    UE_LOG(LogTemp, Log, TEXT("成功解析 %d 个坐标点"), Result.Num());
    return Result;
}

// 设置世界坐标路径点并生成样条线
void AOptimalRouteActor::SetRoutePoints(const TArray<FVector>& WorldPoints)
{
    if (WorldPoints.Num() < 2)
    {
        UE_LOG(LogTemp, Warning, TEXT("路径点数量不足"));
        return;
    }
    SplineComponent->ClearSplinePoints();
    for (int i = 0; i < WorldPoints.Num(); i++)
{
SplineComponent->AddSplinePoint(WorldPoints[i],
ESplineCoordinateSpace::World, false);
    }
    SplineComponent->UpdateSpline();
    BuildSplineMeshes();
UE_LOG(LogTemp, Log, TEXT("样条线已生成，共 %d 个点"), WorldPoints.Num());
}

void AOptimalRouteActor::BuildSplineMeshes()
{
    // 清理旧网格
    for (int i = 0; i < SplineMeshes.Num(); i++)
    {
        if (SplineMeshes[i])
        SplineMeshes[i]->DestroyComponent();
    }
    SplineMeshes.Empty();
    // 加载圆柱体网格
    UStaticMesh* CylinderMesh = LoadObject<UStaticMesh>(nullptr, TEXT("/Engine/BasicShapes/Cylinder.Cylinder"));
    if (!CylinderMesh) return;
    int32 NumPoints = SplineComponent->GetNumberOfSplinePoints();
    if (NumPoints < 2) return;

    // 为每段创建 SplineMeshComponent
    for (int i = 0; i < NumPoints - 1; i++)
{      
USplineMeshComponent*SplineMesh=NewObject<USplineMeshComponent>(this);
        SplineMesh->SetStaticMesh(CylinderMesh);
        SplineMesh->SetMobility(EComponentMobility::Movable);
        SplineMesh->AttachToComponent(SplineComponent, FAttachmentTransformRules::KeepRelativeTransform);
        SplineMesh->RegisterComponent();

FVector StartPos=SplineComponent->GetLocationAtSplinePoint(i, ESplineCoordinateSpace::World);
FVector EndPos=SplineComponent->GetLocationAtSplinePoint(i+1, ESplineCoordinateSpace::World);
        FVector StartTangent = SplineComponent->GetTangentAtSplinePoint(i, ESplineCoordinateSpace::World);
        FVector EndTangent = SplineComponent->GetTangentAtSplinePoint(i + 1, ESplineCoordinateSpace::World);
SplineMesh->SetStartAndEnd(StartPos, StartTangent, EndPos, EndTangent, true);
        SplineMesh->SetStartScale(FVector2D(0.1f, 0.1f));
        SplineMesh->SetEndScale(FVector2D(0.1f, 0.1f));
        SplineMeshes.Add(SplineMesh);
    }
}

```
