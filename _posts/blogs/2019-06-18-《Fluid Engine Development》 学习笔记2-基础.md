---
layout: page
title: 《Fluid Engine Development》 学习笔记2-基础
category: 
    - blogs


---

断断续续花了一个月，终于把这本书的一二两章啃了下来，理解流体模拟的理论似乎不难，无论是《Fluid Simulation for Computer Graphics》还是《计算流体力学基础及其应用》都能很好帮助程序员去理解这些原理，可在缺乏实践情况下，这种对原理的理解其实跟死记硬背没什么区别。《Fluid Engine Development》提供了一个实现完成的流体模拟引擎以及它的编程实现原理，充分帮助程序员通过编程实现流体动画引擎，以此完成流体模拟学习的第一步。这不，早在今年一月就嚷嚷研究学习流体模拟却苦苦挣扎无法入门的我，在抄着[作者的代码](https://github.com/doyubkim/fluid-engine-dev)看着作者的书的情况，终于实现了一个流体模拟引擎。我终于可以自信地说自己已经入门了流体模拟o(╥﹏╥)o。





## 流体模拟

我学习流体模拟的本质原因，是因为迷恋上了复杂的流体动画，渴望通过编程来创造它们。流体模拟主要是指结合流体模拟的物理现象、方程和计算机图形学的方法来模拟海面、海浪、烟雾等场景。

流体模拟的基本方法可分为三类: 基于纹理变换的流体模拟、基于二维高度场网格的流体模拟以及基于真实物理方程的流体模拟。

基于纹理变换的流体模拟只需要对水面纹理进行法向扰动后、绘制水面的倒影(反射)以及绘制水底的情况(透射)即可绘制出一般的水面效果。 但这种方法由于其根本上没有水面网格，所以水面起伏的绘制效果不明显。

基于二维高度场的网格流体模拟方法把水面表示成为一个连续的平面网格，然后生成一系列对应于这张网络的连续的高度纹理-称为高度图。接着每个网格顶点对应于一个高度图的像素，作为水面高度，从而表示出整个水面。

以上2者方法相对简单，但真实度不高，本质上只是hack，基于真实物理方程的流体模拟才能制作目前所能呈现在计算机上最真实的流体动画。尽管这种方法，受限于过高的计算量，一直处于离线渲染领域，很难在实时游戏中使用，但随着实时流体模拟技术的进步以及硬件条件的进步，在游戏中使用这种方法并不遥远，实际上，这甚至已经成为了现实。看看这个[视频](https://vdn1.vzuu.com/SD/962fbf2e-5008-11e8-8f62-0242ac112a13.mp4?disable_local_cache=1&bu=com&expiration=1560770247&auth_key=1560770247-0-0-be614aa4ddf6be8a59a11ffbf7be0104&f=mp4&v=hw)，实时流体模拟已经能做得很好了。

流体模拟的基本方法主要有三种，一种使用粒子（拉格朗日方法），一种使用网格（欧拉方法），第三种则是2者的混合。本文主要讲基于粒子的流体模拟。



## 基于物理的动画

流体动画首先是动画，动画的本质是在给定时间序列播放一系列图像，由此我们可以编写帧的结构体和动画的抽象类。我们创建新的动画时，只要编写类继承Animation，再重写onUpdate函数，在指定帧更新图像即可。

```c++
struct Frame final {
	public:
		int index = 0;
		double timeIntervalInSeconds = 1.0 / 60.0;
		double GetTimeInSeconds() const;
		void Advance();
		void Advance(int delta);
};
	
class Animation{
public:
	void Update(const Frame& frame);
protected:

	//**********************************************
	//the function is called from Animation:Update();
	//this function and implement its logic for updating the animation state.
	//**********************************************
	virtual void onUpdate(const Frame& frame) = 0;
};

void Animation::Update(const Frame & frame){
	onUpdate(frame);
}

double Frame::GetTimeInSeconds() const{
	return index * timeIntervalInSeconds;
}

void Frame::Advance(){
	++index;
}

void Frame::Advance(int delta){
	index += delta;
}
```
流体动画属于基于物理的动画，它不同于正常的动画，正常的动画并不依赖外界的输入以及其他帧的数据和状态，它们大多是根据时间状态的改变来播放不同的图像，或是通过以时间为输入的函数（例如sin(t)图像）来切换状态。基于物理的动画正好相反，当前帧的数据和状态完全是由上一帧决定的。由此编写了PhysicsAnimation类

```c++
class PhysicsAnimation : public Animation{
	public:
		double GetCurrentTime() const { return _currentTime; }
	private:
		void initialize();
		void timeStep(double timeIntervalInSeconds);

        Frame _currentFrame;
        double _currentTime = 0.0;
protected:
	virtual void onUpdate(const Frame& frame) override final;

	//**********************************************
	//the function is called from Animation:timeStep(double);
	//This function is called for each time-step;
	//**********************************************
	virtual void onTimeStep(double timeIntervalInSeconds) = 0;

	//**********************************************
	//the function is called from Animation:initialize();
	//Called at frame 0 to initialize the physics state.;
	//**********************************************
	virtual void onInitialize() = 0;
};

void PhysicsAnimation::initialize(){
	onInitialize();
}


void PhysicsAnimation::onUpdate(const Frame & frame){
	if (frame.index > _currentFrame.index) {
		if (_currentFrame.index < 0) {
			initialize();
		}

		int numberOfFrames = frame.index - _currentFrame.index;

		for (int i = 0; i < numberOfFrames; ++i) {
			timeStep(frame.timeIntervalInSeconds);
		}

		_currentFrame = frame;
	}
}
```
我们可以看到PhysicsAnimation重写了onUpdate函数，采用渐近更新每一帧



## 粒子动画系统

我们使用粒子模拟流体，那就不可避免的要使用粒子系统

以下是我个人流体引擎粒子系统动画系统的部分头文件

```c++
class ParticleSystemData3
{
    public:
    typedef std::vector<double> ScalarData;
    typedef std::vector<Vector3D> VectorData;

    ParticleSystemData3();
    virtual ~ParticleSystemData3();
    explicit ParticleSystemData3(size_t numberOfParticles);
    void Resize(size_t newNumberOfParticles);
    size_t GetNumberOfParticles() const;
    size_t AddVectorData(const Vector3D& initialVal = Vector3D::zero);
    size_t AddScalarData(double initialVal = 0.0);
    const std::vector<Vector3D>& GetPositions() const;
    std::vector<Vector3D>& GetPositions();
    const std::vector<Vector3D>& GetVelocities() const;
    std::vector<Vector3D>& GetVelocities();
    const std::vector<Vector3D>& GetForces() const;
    std::vector<Vector3D>& GetForces();
    void AddParticle(
        const Vector3D& newPosition,
        const Vector3D& newVelocity,
        const Vector3D& newForce = Vector3D::zero);
    void AddParticles(
        const std::vector<Vector3D>& newPositions,
        const std::vector<Vector3D>& newVelocities,
        const std::vector<Vector3D>& newForces);
    const std::vector<double>& ScalarDataAt(size_t idx) const;
    std::vector<double>& ScalarDataAt(size_t idx);
    const std::vector<Vector3D>& VectorDataAt(size_t idx) const;
    std::vector<Vector3D>& VectorDataAt(size_t idx);

    double GetParticleRadius() const;
    void SetParticleRadius(double newRadius);
    double GetParticleMass() const;
    virtual void SetParticleMass(double newMass);

    void BuildNeighborSearcher(double maxSearchRadius);
    void BuildNeighborLists(double maxSearchRadius);

    const std::shared_ptr<PointNeighborSearcher3> & GetNeighborSearcher() const;
    const std::vector<std::vector<size_t>>& GetNeighborLists() const;
    protected:
    size_t _positionIdx;
    size_t _velocityIdx;
    size_t _forceIdx;
    size_t _numberOfParticles = 0;
    double _radius = 1e-3;
    double _mass = 1e-3;

    std::vector<ScalarData> _scalarDataList;
    std::vector<VectorData> _vectorDataList;
    std::shared_ptr<PointNeighborSearcher3> _neighborSearcher;
    std::vector<std::vector<size_t>>_neighborLists;
};

class ParticleSystemSolver3 : public PhysicsAnimation{
private:
    void timeStepStart(double timeStepInSeconds);
    void timeStepEnd(double timeStepInSeconds);
    void timeIntegration(double timeIntervalInSeconds);
    void resolveCollision();
    void updateCollider(double timeStepInSeconds);
    void updateEmitter(double timeStepInSeconds);
    ParticleSystemData3::VectorData _newPositions;
    ParticleSystemData3::VectorData _newVelocities;
protected:
    void onTimeStep(double timeIntervalInSeconds) override;
    virtual void onInitialize() override;
	//**********************************************
	//the function is called in ParticleSystemSolver3:timeStepStart(double);
	// Called when a time-step is about to begin;
	//**********************************************
	virtual void onTimeStepStart(double timeStepInSeconds);

	//**********************************************
	//the function is called in ParticleSystemSolver3:timeStepEnd(double);
	// Called when a time-step is about to end;
	//**********************************************
	virtual void onTimeStepEnd(double timeStepInSeconds);

	//**********************************************
	//the function is called in ParticleSystemSolver3:onTimeStep(double);
	//accumulate forces
	//**********************************************
	virtual void accumulateForces(double timeIntervalInSeconds);

	std::shared_ptr<ParticleSystemData3> _particleSystemData;
	std::shared_ptr<VectorField3> _wind;
    Vector3D _gravity = Vector3D(0.0, -9.8, 0.0);
	double _dragCoefficient = 1e-4;
	std::shared_ptr<Collider3> _collider;
	double _restitutionCoefficient = 0.0;
	std::shared_ptr<ParticleEmitter3> _emitter;
};
```
主要看粒子系统重写的onTimeStep函数，它用以更新粒子系统，它依次调用了5个函数。

accumulateForces用以累加作用于粒子上的力，timeIntegration用以更新粒子速度和位置.resolveCollision处理碰撞，timeStepStart，timeStepEnd分别用作每帧逻辑更新前后的事件调用以及内部数据更新

```c++
void ParticleSystemSolver3::onTimeStep(double timeIntervalInSeconds)
{
	timeStepStart(timeIntervalInSeconds);

	accumulateForces(timeIntervalInSeconds);
	timeIntegration(timeIntervalInSeconds);
	resolveCollision();

	timeStepEnd(timeIntervalInSeconds);
}
```
具体实现如下

```c++
void CalfFluidEngine::ParticleSystemSolver3::timeStepStart(double timeStepInSeconds)
{
	auto& forces = _particleSystemData->GetForces();
    tbb::parallel_for(
        tbb::blocked_range<size_t>(0, forces.size()),
        [&](const tbb::blocked_range<size_t> & b) {
        for (size_t i = b.begin(); i != b.end(); ++i)
            forces[i] = Vector3D::zero;
    });

    updateCollider(timeStepInSeconds);
    updateEmitter(timeStepInSeconds);

    size_t n = _particleSystemData->GetNumberOfParticles();
    _newPositions.resize(n);
    _newVelocities.resize(n);

    onTimeStepStart(timeStepInSeconds);
}

void CalfFluidEngine::ParticleSystemSolver3::accumulateForces(double timeIntervalInSeconds)
{
	size_t n = _particleSystemData->GetNumberOfParticles();
	auto& forces = _particleSystemData->GetForces();
	auto& velocities = _particleSystemData->GetVelocities();
	auto& positions = _particleSystemData->GetPositions();
	const double mass = _particleSystemData->GetParticleMass();

	tbb::parallel_for(
		tbb::blocked_range<size_t>(0, n),
		[&](const tbb::blocked_range<size_t> & b) {
			for (size_t i = b.begin(); i != b.end(); ++i){
				// Gravity
				Vector3D force = mass * _gravity;

				// Wind forces
				Vector3D relativeVel = velocities[i] - _wind->Sample(positions[i]);
				force += -_dragCoefficient * relativeVel;

				forces[i] += force;
		}
		
	});
}

void CalfFluidEngine::ParticleSystemSolver3::timeIntegration(double timeIntervalInSeconds)
{
	size_t n = _particleSystemData->GetNumberOfParticles();
	auto& forces = _particleSystemData->GetForces();
	auto& velocities = _particleSystemData->GetVelocities();
	auto& positions = _particleSystemData->GetPositions();
	const double mass = _particleSystemData->GetParticleMass();

	tbb::parallel_for(
		tbb::blocked_range<size_t>(0, n),
		[&](const tbb::blocked_range<size_t> & b) {
		for (size_t i = b.begin(); i != b.end(); ++i) {
			Vector3D& newVelocity = _newVelocities[i];
			newVelocity = velocities[i];
			newVelocity.AddScaledVector(forces[i] / mass, timeIntervalInSeconds);

			Vector3D& newPosition = _newPositions[i];
			newPosition = positions[i];
			newPosition.AddScaledVector(newVelocity, timeIntervalInSeconds);
		}

	});
}

void CalfFluidEngine::ParticleSystemSolver3::resolveCollision()
{
	if (_collider != nullptr) {
		size_t numberOfParticles = _particleSystemData->GetNumberOfParticles();
		const double radius = _particleSystemData->GetParticleRadius();

		tbb::parallel_for(
			tbb::blocked_range<size_t>(size_t(0), numberOfParticles),
			[&](const tbb::blocked_range<size_t> & b) {
			for (size_t i = b.begin(); i != b.end(); ++i)
			{
				_collider->ResolveCollision(
					radius,
					_restitutionCoefficient,
					&_newPositions[i],
					&_newVelocities[i]);
			}
				
		});
	}
}

void CalfFluidEngine::ParticleSystemSolver3::timeStepEnd(double timeStepInSeconds)
{
	size_t n = _particleSystemData->GetNumberOfParticles();
	auto& positions = _particleSystemData->GetPositions();
	auto& velocities = _particleSystemData->GetVelocities();

	tbb::parallel_for(
		tbb::blocked_range<size_t>(0, n),
		[&](const tbb::blocked_range<size_t> & b) {
		for (size_t i = b.begin(); i != b.end(); ++i)
		{
			positions[i] = _newPositions[i];
			velocities[i] = _newVelocities[i];
		}
			
	});

	onTimeStepEnd(timeStepInSeconds);
}
```
## 碰撞检测和处理

上一节我们提到了碰撞，这个需要特别列出一节来讲。

碰撞在物理模拟乃至物理引擎领域都是一个非常复杂的问题，《Fluid Engine Development》对于这一块并没有进行深入，碰撞检测只是简单检测碰撞点是否在几何表面的内部，以及碰撞点，粒子的中心间的距离是否小于粒子的半径。

至于碰撞的处理，主要是改变粒子的速度，和把粒子修正到正确的位置。

粒子的正确位置为碰撞点+碰撞点法线 * 粒子半径,也就是紧贴碰撞面表面的位置

至于粒子的速度，这里把粒子相对碰撞面的速度拆解切向速度和法向速度

经过碰撞，法向速度变更为 $- R V_{n}​$ （R是恢复系数，Vn是法向速度）

切向速度变更为    $max\left( 1 - u  \frac{\left|\Delta V_{n}\right|}{V_{t}} \right)  V_{t}$

切向速度的变更大家或许会有疑惑，这里列出推导过程

假定$\Delta V_{n} = V_{n}^{new} - V_{n} = (- R - 1)V_{n}​$

已知$\Delta V_{t} =  a_{t}\Delta t = F_{f}/m \Delta t = u F_{n}/m  \Delta t​$

又上类推得$\Delta V_{n} =  a_{n}\Delta t = F_{n}/m \Delta t$

所以$\Delta V_{t} = u \Delta V_{n}​$

因为新切向速度必然小于原速度，所以$\Delta V_{t} = min(u\Delta V_{n},V_{t})​$

可得 $V_{t}^{new} =max\left( 1 - u \frac{\left|\Delta V_{n}\right|}{V_{t}} \right)  V_{t}$

具体代码实现如下

```c++
struct ColliderQueryResult final {
			double distance;
			Vector3D point;
			Vector3D normal;
			Vector3D velocity;
};

void CalfFluidEngine::Collider3::ResolveCollision(
	double radius, 
	double restitutionCoefficient, 
	Vector3D * position, 
	Vector3D * velocity)
{
	ColliderQueryResult res;
	getClosestPoint(_surface, *position, &res);

	if (isPenetrating(res, *position, radius))
	{
		Vector3D targetNormal = res.normal;
		Vector3D targetPoint = res.point + radius * targetNormal;
		Vector3D colliderVelAtTargetPoint = res.velocity;

		Vector3D relativeVel = *velocity - colliderVelAtTargetPoint;
		double normalDotRelativeVel = Vector3D::Dot(targetNormal, relativeVel);
		Vector3D relativeVelN = normalDotRelativeVel * targetNormal;
		Vector3D relativeVelT = relativeVel - relativeVelN;

		if (normalDotRelativeVel < 0.0) {
			Vector3D deltaRelativeVelN =
				(-restitutionCoefficient - 1.0) * relativeVelN;
			relativeVelN *= -restitutionCoefficient;

			// Apply friction to the tangential component of the velocity
			// From Bridson et al., Robust Treatment of Collisions, Contact and
			// Friction for Cloth Animation, 2002
			// http://graphics.stanford.edu/papers/cloth-sig02/cloth.pdf
			if (relativeVelT.SquareMagnitude() > 0.0) {
				double frictionScale = std::max(
					1.0 - _frictionCoeffient * deltaRelativeVelN.Magnitude() /
					relativeVelT.Magnitude(),
					0.0);
				relativeVelT *= frictionScale;
			}

			*velocity =
				relativeVelN + relativeVelT + colliderVelAtTargetPoint;
		}

		*position = targetPoint;
	}
}

bool CalfFluidEngine::Collider3::isPenetrating(
	const ColliderQueryResult & colliderPoint, 
	const Vector3D & position, 
	double radius)
{
	return _surface->IsInside(position) ||
		colliderPoint.distance < radius;
}
```

## 粒子发射器

粒子系统有一个名为updateEmitter的函数需要注意一下，它在timeStepStart函数和onInitialize()函数中被调用，用于随机生成粒子系统的粒子数据。不过在实例demo中，粒子只生成了一次，毕竟每帧重新生成粒子是非常耗时的。粒子发射器的核心代码如下。

```c++
void CalfFluidEngine::VolumeParticleEmitter3::onUpdate(double currentTimeInSeconds, double timeIntervalInSeconds)
{
	auto particles = GetTarget();

	if (particles == nullptr) {
		return;
	}

	if (_numberOfEmittedParticles > 0 && _isOneShot) {
		return;
	}

	std::vector<Vector3D> newPositions;
	std::vector<Vector3D> newVelocities;
	std::vector<Vector3D> Forces;
	emit(particles, &newPositions, &newVelocities);

	particles->AddParticles(newPositions, newVelocities, Forces);
}

void CalfFluidEngine::VolumeParticleEmitter3::emit(const std::shared_ptr<ParticleSystemData3>& particles, std::vector<Vector3D>* newPositions, std::vector<Vector3D>* newVelocities)
{
	if (_surface == nullptr) return;
	_surface->Update();

	BoundingBox3D region = _bounds;
	if (_surface->IsBounded()) {
		BoundingBox3D surfaceBox = _surface->GetBoundingBox();
		region.lowerCorner = max(region.lowerCorner, surfaceBox.lowerCorner);
		region.upperCorner = min(region.upperCorner, surfaceBox.upperCorner);
	}

	const double j = GetJitter();
	const double maxJitterDist = 0.5 * j * _spacing;

	if (_allowOverlapping || _isOneShot) 
	{
		_pointGenerator->ForEachPoint(region, _spacing, [&](const Vector3D& point) {
			Vector3D randomDir = uniformSampleSphere(random(_rng), random(_rng));
			Vector3D offset = maxJitterDist * randomDir;
			Vector3D candidate = point + offset;
			if (_surface->SignedDistance(candidate) <= 0.0) {
				if (_numberOfEmittedParticles < _maxNumberOfParticles) {
					newPositions->push_back(candidate);
					++_numberOfEmittedParticles;
				}
				else {
					return false;
				}
			}

			return true;
		});
	}
	else 
	{
		PointHashGridSearcher3 neighborSearcher(
			Vector3<size_t>(
				kDefaultHashGridResolution, 
				kDefaultHashGridResolution,
				kDefaultHashGridResolution),
			2.0 * _spacing);
		if (!_allowOverlapping) {
			neighborSearcher.Build(particles->GetPositions());
		}

		_pointGenerator->ForEachPoint(region, _spacing, [&](const Vector3D& point) {
			Vector3D randomDir = uniformSampleSphere(random(_rng), random(_rng));
			Vector3D offset = maxJitterDist * randomDir;
			Vector3D candidate = point + offset;
			if (_surface->SignedDistance(candidate) <= 0.0 &&
				(!_allowOverlapping &&!neighborSearcher.HasNearbyPoint(candidate, _spacing))) {
				if (_numberOfEmittedParticles < _maxNumberOfParticles) {
					newPositions->push_back(candidate);
					neighborSearcher.Add(candidate);
					++_numberOfEmittedParticles;
				}
				else {
					return false;
				}
			}

			return true;
		});
	}

	newVelocities->resize(newPositions->size());

	tbb::parallel_for(
		tbb::blocked_range<size_t>(0, newVelocities->size()),
		[&](const tbb::blocked_range<size_t> & b) {
		for (size_t i = b.begin(); i != b.end(); ++i)
			(*newVelocities)[i] = velocityAt((*newPositions)[i]);
	});
}

Vector3D CalfFluidEngine::VolumeParticleEmitter3::velocityAt(const Vector3D & point) const
{
	Vector3D r = point - _surface->transform.GetTranslation();
	return _linearVel + Vector3D::Cross(_angularVel,r) + _initialVel;
}

void CalfFluidEngine::BccLatticePointGenerator::ForEachPoint(const BoundingBox3D & boundingBox, double spacing, const std::function<bool(const Vector3D&)>& callback) const
{
	double halfSpacing = spacing / 2.0;
	double boxWidth = boundingBox.GetWidth();
	double boxHeight = boundingBox.GetHeight();
	double boxDepth = boundingBox.GetDepth();

	Vector3D position;
	bool hasOffset = false;
	bool shouldQuit = false;
	for (int k = 0; k * halfSpacing <= boxDepth && !shouldQuit; ++k) {
		position.z = k * halfSpacing + boundingBox.lowerCorner.z;

		double offset = (hasOffset) ? halfSpacing : 0.0;

		for (int j = 0; j * spacing + offset <= boxHeight && !shouldQuit; ++j) {
			position.y = j * spacing + offset + boundingBox.lowerCorner.y;

			for (int i = 0; i * spacing + offset <= boxWidth; ++i) {
				position.x = i * spacing + offset + boundingBox.lowerCorner.x;
				if (!callback(position)) {
					shouldQuit = true;
					break;
				}
			}
		}

		hasOffset = !hasOffset;
	}
}
```


## 邻域粒子搜索

我们看到上节中粒子发射器的代码实现里使用了PointHashGridSearcher3这个类，这是我为什么说粒子发生器需要注意的原因，这是个非常精妙的算法，它将3D点映射到整数，它被这样设计的意义是为了加速搜索。



它将长方体的包围盒区域划分成多个长方形块，将3D点的坐标除以长方形块的边长，将它转换成三维整形数（Vector3<size_t>），再将三维整数转换成普通整形数作为3D坐标的哈希值。

代码如下

```c++
size_t CalfFluidEngine::PointHashGridSearcher3::GetHashKeyFromBucketIndex(const Vector3<size_t>& bucketIndex) const {
	Vector3<size_t> wrappedIndex = bucketIndex;
	wrappedIndex.x = bucketIndex.x % _resolution.x;
	wrappedIndex.y = bucketIndex.y % _resolution.y;
	wrappedIndex.z = bucketIndex.z % _resolution.z;
	return (wrappedIndex.z 
		* _resolution.y + wrappedIndex.y) 
		* _resolution.x + wrappedIndex.x;
}

Vector3<size_t> CalfFluidEngine::PointHashGridSearcher3::GetBucketIndex(const Vector3D & position) const
{
	Vector3<size_t> bucketIndex;
	bucketIndex.x = static_cast<size_t>(
		std::floor(position.x / _gridSpacing));
	bucketIndex.y = static_cast<size_t>(
		std::floor(position.y / _gridSpacing));
	bucketIndex.z = static_cast<size_t>(
		std::floor(position.z / _gridSpacing));
	return bucketIndex;
}
```

PointHashGridSearcher3内部有一个二维数组，或者可以称它为桶数组，桶数组的键值就是由3D坐标转换而来的key，代表一个包围盒区域内的长方形块。桶数组存储所对应长方块内的所有粒子在粒子数组中的索引。

当需要查找邻域粒子时，就可以通过粒子坐标所对应的key值查找获取粒子所在长方块附近其他长方形块的key，从而获取粒子所在长方形块和粒子附近长方形块内其他粒子。

获取附近key值代码实现如下

```c++
void CalfFluidEngine::PointHashGridSearcher3::getNearbyKeys(const Vector3D & position, size_t * nearbyKeys) const
{
	Vector3<size_t> originIndex = GetBucketIndex(position), nearbyBucketIndices[8];

	for (int i = 0; i < 8; i++) {
		nearbyBucketIndices[i] = originIndex;
	}

	if ((originIndex.x + 0.5f) * _gridSpacing <= position.x) {
		nearbyBucketIndices[4].x += 1;
		nearbyBucketIndices[5].x += 1;
		nearbyBucketIndices[6].x += 1;
		nearbyBucketIndices[7].x += 1;
	}
	else {
		nearbyBucketIndices[4].x -= 1;
		nearbyBucketIndices[5].x -= 1;
		nearbyBucketIndices[6].x -= 1;
		nearbyBucketIndices[7].x -= 1;
	}

	if ((originIndex.y + 0.5f) * _gridSpacing <= position.y) {
		nearbyBucketIndices[2].y += 1;
		nearbyBucketIndices[3].y += 1;
		nearbyBucketIndices[6].y += 1;
		nearbyBucketIndices[7].y += 1;
	}
	else {
		nearbyBucketIndices[2].y -= 1;
		nearbyBucketIndices[3].y -= 1;
		nearbyBucketIndices[6].y -= 1;
		nearbyBucketIndices[7].y -= 1;
	}

	if ((originIndex.z + 0.5f) * _gridSpacing <= position.z) {
		nearbyBucketIndices[1].z += 1;
		nearbyBucketIndices[3].z += 1;
		nearbyBucketIndices[5].z += 1;
		nearbyBucketIndices[7].z += 1;
	}
	else {
		nearbyBucketIndices[1].z -= 1;
		nearbyBucketIndices[3].z -= 1;
		nearbyBucketIndices[5].z -= 1;
		nearbyBucketIndices[7].z -= 1;
	}

	for (int i = 0; i < 8; i++) {
		nearbyKeys[i] = GetHashKeyFromBucketIndex(nearbyBucketIndices[i]);
	}

}


```

获取粒子附近其他粒子代码如下

```c++
size_t nearbyKeys[8];
getNearbyKeys(origin, nearbyKeys);

const double queryRadiusSquared = radius * radius;

for (int i = 0; i < 8; i++) {
    const auto& bucket = _buckets[nearbyKeys[i]];
    size_t numberOfPointsInBucket = bucket.size();

    for (size_t j = 0; j < numberOfPointsInBucket; ++j) {
        size_t pointIndex = bucket[j];
        double rSquared = (_points[pointIndex] - origin).SquareMagnitude();
        if (rSquared <= queryRadiusSquared) {
            ......
        }
    }
}
```

## 矢量算子

下一节将进入流体力学的部分，在那之前先重温或学习一下关于矢算场的数学知识，如果不了解的话，会对Navier-Stocks方程的一些数学符号感到疑惑。

《Fluid Engine Development》有介绍这部分的知识，更详细的大家可以阅读另一位[博主](https://www.cnblogs.com/yaoyansi/articles/1891015.html)推荐的《失算场论札记》,不过这本书我淘宝也买不到，在学校图书馆借了看，后来花9块钱买了[矢量分析与场论-工程数学-第四版](https://book.douban.com/subject/10752782/)

### 偏导数

$\frac{\partial f}{\Delta } = \frac{f(x + \Delta,y,z) - f(x,y,z)}{ \Delta}​$ 

### 梯度

$\nabla f(x,y,z) = (\frac{\partial f}{\partial x} , \frac{\partial f}{\partial y} , \frac{\partial f}{\partial z}  )​$

$\nabla​$ 是梯度算子，用以测量标量的变化率和变换方向，

梯度是标量场增长最快的方向，梯度的长度是这个最大的变化率

梯度本意是一个向量（矢量），当某一函数在某点处沿着该方向的方向导数取得该点处的最大值，即函数在该点处沿方向变化最快，变化率最大（为该梯度的模）

对程序开发者而言，它是由一阶偏导组成的矢量

当我们需要知道函数在特定方向上的变化率，通过将梯度与该方向的向量点乘，我们就可以计算出函数在这一点的方向导数 $\frac{\partial f}{\partial n} =\nabla f  \cdot n ​$

### 散度

$\nabla \cdot f(x) = (\frac{\partial }{\partial x} , \frac{\partial }{\partial y} , \frac{\partial }{\partial z}  ) \cdot  f(x)​$

$\nabla \cdot ​$ 是散度算子，散度就是通量密度，通量即通过一个面的某物理量

在流体力学中，速度场的散度是体积膨胀率，散度(divergence)算子用于描述向量场中在某点的聚集或发散程度

对程序开发者，这就是梯度算子与向量场的点乘

流体模拟中，流体往往有着难以压缩的性质，通常认为速度场的散度为零

### 旋度

$\nabla\times f(x) = (\frac{\partial }{\partial x} , \frac{\partial }{\partial y} , \frac{\partial }{\partial z}  ) \times  f(x)​$

$\nabla\times​$ 是旋度算子，旋度就是环量密度，以水旋涡为例，沿着圆周关于速度的线积分就是环量，环量除以圆面积，然后让面积无限小，比值就是旋度，旋度用以描述某个点的旋转程度

对程序开发者，这就是梯度算子与向量场的叉积

### 拉普拉斯算子

$\nabla^{2} f(x) = \nabla \cdot \nabla f(x) =\frac{\partial^{2} f}{\partial x^{2} } +\frac{\partial^{2} f}{\partial y^{2} } +\frac{\partial^{2}  f}{\partial z^{2} } ​$

$\nabla^{2} $是拉普拉斯算子，又称拉氏算子，它的本质是梯度的散度

拉普拉斯算子通常用于描述一个函数中某点的值与它邻域的平均值之差，或者说用于描述某点 在它的邻域中是“多么与众不同”

拉普拉斯算子可以检测图像的边缘和峰值，也可以用以模糊矢量场的尖锐部分（拉普拉斯算子 加上原始矢量场）

## 流体运动的基本原理

现在开始进入正题，前面讲的都是编程相关的细节，现在开始流体力学的部分。

流体运动的本质是源于多种力的组合。

在保证密度不变的情况下，流体受到重力，压力，粘度力三种力的作用。

重力不需要我太多介绍，主要是压力和粘度力

### 压力和压力梯度力

风是高压区吹到低压区的，同样的规则适用于流体。

压力其实是压力梯度力，当你潜水时，随着深度的增加，收到的压力也随之增加。这种压力差会沿着深度在与重力相反的方向产生向上的压力。正因为这种力，水可以保持其状态而不是收缩

假设一个立方体大小的流体，单位大小是L，密度为 $\rho​$， 假设压力沿x轴方向，左右之间的压力差是$\Delta p​$，压力大小就等于$\Delta p L^{2}​$，所以 $F_{p} = -\Delta p L^{2}= ma_{p} = L^{3}\rho a_{p} ​$

可推导得 $a_{p} = \frac{-\Delta p}{\rho L}​$，假定立方体非常小，可得 $a_{p} = -\frac{\partial p}{\partial x} \frac{1}{\rho} ​$

接着我们将它推导到三维上 ，可得 $a_{p} = -\frac{\nabla p}{\rho}  ​$

所以$ a = g  -\frac{\nabla p}{\rho} + \cdots​$

### 粘度

流体模拟中粘度类似于阻尼，它试图模糊2点间的速度差异，它导致液体有了一定的粘稠感。正好，拉普拉斯算子能做到模糊矢量场。

$V_{new} = V + \Delta t u \nabla^{2}V​$   u 是粘度系数 

由上式可以得到粘度影响的加速度$ a_{V} =u \nabla^{2}V​$

所以$ a = g  -\frac{\nabla p}{\rho} + u \nabla^{2}V$ 这就是 著名的 纳维斯托克斯(Navier--Stokes, NS)方程

### 

