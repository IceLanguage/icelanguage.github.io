---
layout: page
title: 《Fluid Engine Development》 学习笔记4-预测校正不可压缩SPH-PCISPH
category: 
    - blogs


---

传统SPH方案的主要问题之一是时间步长限制。在原始的SPH中，我们首先从当前设置计算密度，使用EOS计算压强，应用压力梯度，然后运行时间积分。这个过程意味着只需要一定的压缩量就可以触发内核半径内的压力，从而延迟计算。因此，我们需要使用更小的时间步长(意味着更多的迭代)，这在计算上是昂贵的。或者，我们可以使用不那么严格的EOS，然而，这个解决方案可能会引入类似弹簧的振荡。微调参数如声速或粘度可以帮助避免此类问题。然而，这并不是一个基本的解决方案，对用户来说也是不切实际的。Solenthaler 和Pajarola通过在SPH模拟中引入预测-校正器概念来解决这个问题。这种又称为预测校正不可压缩SPH（PCISPH），它是一种误差测量算法，假定测量值和期望密度的差值是误差。本篇文章总结我在《Fluid Engine Development》学到关于PCISPH的知识。



## 算法原理

1.计算合力，预测速度位置，碰撞检测或碰撞处理

2.计算密度后测量密度误差，利用密度误差更新压强

3.计算新压强梯度力

4.多次重复以上过程，得到使密度误差最小的修正力（压强梯度力），然后使用累积力进行下一步

由于它是通过累积校正力使最终状态处于不可压缩状态，而不是通过多个SPH步骤保证不可压缩，所以它不需要在意时间步长的问题

算法代码实现结构如下，

```c++
void CalfFluidEngine::PCISPHSolver3::accumulatePressureForce(double timeStepInSeconds)
{   
	auto particles = GetSphData();
	const size_t numberOfParticles = particles->GetNumberOfParticles();
	const double delta = computeDelta(timeStepInSeconds);
	const double targetDensity = particles->GetDensity();
	const double mass = particles->GetParticleMass();

	auto& p = particles->GetPressures();
	auto& d = particles->GetDensities();
	auto& x = particles->GetPositions();
	auto& v = particles->GetVelocities();
	auto& f = particles->GetForces();

	//Initialize 

	for (unsigned int k = 0; k < _maxNumberOfIterations; ++k) 
	{
		// Predict velocity and position

		// Resolve collisions

		// Compute pressure from density error

		// Compute pressure gradient force 

		// Compute max density error
		double maxDensityError = ......;
		double densityErrorRatio = maxDensityError / targetDensity;


		if (std::fabs(densityErrorRatio) < _maxDensityErrorRatio){
			break;
		}
	}

	//Accumlate pressure force
}
```

我们可以看到预测-校正本身是一个循环体，循环在密度误差小于指定阈值循环执行次数小于指定迭代次数时终止

而循环体内所做的就是不断进行修正压力，直到密度误差足够小



需要注意的是在执行这套算法前，粒子所受其他力包括粘度，重力已经全部计算完成，已全部累加到forces数组中

```c++
void accumulateForces(double timeIntervalInSeconds)
{
	ParticleSystemSolver3::accumulateForces(timeIntervalInSeconds);
	accumulateViscosityForce();
	accumulatePressureForce(timeIntervalInSeconds);
}
```

这边给出完整的算法实现代码

```c++
void CalfFluidEngine::PCISPHSolver3::accumulatePressureForce(double timeStepInSeconds)
{
	auto particles = GetSphData();
	const size_t numberOfParticles = particles->GetNumberOfParticles();
	const double delta = computeDelta(timeStepInSeconds);
	const double targetDensity = particles->GetDensity();
	const double mass = particles->GetParticleMass();

	auto& p = particles->GetPressures();
	auto& d = particles->GetDensities();
	auto& x = particles->GetPositions();
	auto& v = particles->GetVelocities();
	auto& f = particles->GetForces();

	//Initialize 
	std::vector<double> predictedDensities(numberOfParticles, 0.0);

	SphStandardKernel3 kernel(particles->GetKernelRadius());

	tbb::parallel_for(
		tbb::blocked_range<size_t>(0, numberOfParticles),
		[&](const tbb::blocked_range<size_t> & b) {
		for (size_t i = b.begin(); i != b.end(); ++i)
		{
			p[i] = 0.0;
			_pressureForces[i] = Vector3D::zero;
			_densityErrors[i] = 0.0;
			predictedDensities[i] = d[i];
		}
	});

	for (unsigned int k = 0; k < _maxNumberOfIterations; ++k) 
	{
		// Predict velocity and position
		tbb::parallel_for(
			tbb::blocked_range<size_t>(0, numberOfParticles),
			[&](const tbb::blocked_range<size_t> & b) {
			for (size_t i = b.begin(); i != b.end(); ++i)
			{
				_tempVelocities[i] = v[i] + 
					(f[i] + _pressureForces[i]) / mass * timeStepInSeconds;
				_tempPositions[i] = x[i] + 
					 _tempVelocities[i] * timeStepInSeconds;
			}
		});

		// Resolve collisions
		ParticleSystemSolver3::resolveCollision(_tempPositions, _tempVelocities);

		// Compute pressure from density error
		tbb::parallel_for(
			tbb::blocked_range<size_t>(0, numberOfParticles),
			[&](const tbb::blocked_range<size_t> & b) {
			for (size_t i = b.begin(); i != b.end(); ++i)
			{
				double weightSum = 0.0;
				const auto& neighbors = particles->GetNeighborLists()[i];

				for (size_t j : neighbors) {
					double dist = Vector3D::Distance(_tempPositions[j], _tempPositions[i]);
					weightSum += kernel(dist);
				}
	
				weightSum += kernel(0);

				double density = mass * weightSum;
				double densityError = (density - targetDensity);
				double pressure = delta * densityError;

				if (pressure < 0.0) {
					pressure *= _negativePressureScale;
					densityError *= _negativePressureScale;
				}

				p[i] += pressure;
				predictedDensities[i] = density;
				_densityErrors[i] = densityError;
			}
		});

		// Compute pressure gradient force 
		tbb::parallel_for(
			tbb::blocked_range<size_t>(0, numberOfParticles),
			[&](const tbb::blocked_range<size_t> & b) {
			for (size_t i = b.begin(); i != b.end(); ++i)
			{
				_pressureForces[i] = Vector3D::zero;
			}
		});
		
		SphSystemSolver3::accumulatePressureForce(x, predictedDensities, p, _pressureForces);

		// Compute max density error
		double maxDensityError = 0.0;
		for (size_t i = 0; i < numberOfParticles; ++i) {
			maxDensityError = AbsMax(maxDensityError, _densityErrors[i]);
		}
		double densityErrorRatio = maxDensityError / targetDensity;


		if (std::fabs(densityErrorRatio) < _maxDensityErrorRatio){
			break;
		}
	}

	//Accumlate pressure force
	tbb::parallel_for(
		tbb::blocked_range<size_t>(0, numberOfParticles),
		[&](const tbb::blocked_range<size_t> & b) {
		for (size_t i = b.begin(); i != b.end(); ++i)
		{
			f[i] += _pressureForces[i];
		}
	});
}
```

乍一看算法实现并不难，大部分原理都已经在之前的笔记提过，唯一需要额外关注的PCISPH利用密度误差更新压强的数学公式

## 压强更新公式

压强通过 $p_{i}(t) += \delta \rho_{error}​$更新，这个公式是怎么来的呢，我们来推导一下。



首先回顾一下密度的计算公式 $\rho = \sum_{j} m_{j}W(x_{ij},h)​$  其中 $W(x_{ij},h)​$是光滑核函数,$x_{ij}是x_{i},x_{j}间的距离​$，因为粒子质量一致，所以 $\rho = m\sum_{j} W(x_{ij})​$ 

设当前时间为t，则 $\rho_{t+\Delta t} = m\sum_{j} W(x_{i}(t+\Delta t) -  x_{j}(t+\Delta t)) = m\sum_{j} W(x_{i}(t) +\Delta x_{i}(t) -  x_{j}(t) - \Delta x_{j}(t) ) ​$

设 $r_{ij}(t) = x_{i}(t) - x_{j}(t),\Delta r_{ij} =  \Delta x_{i}(t)  - \Delta x_{j}(t) ​$, 则 $\rho_{t+\Delta t} = m\sum_{j} W(r_{ij} (t) + \Delta r_{ij}(t)) ​$

然后泰勒展开可得 $\rho_{t+\Delta t} = m\sum_{j} W(r_{ij} (t) )+ \nabla W(r_{ij}(t)) \cdot \Delta r_{ij}(t) = \rho_{i}(t) + \Delta \rho_{i}(t)​$
可求得密度增量为$\Delta \rho_{i}(t) =m \sum_{j}\nabla W(r_{ij}(t))  \cdot \Delta r_{ij}(t)= m \sum_{j} \nabla W(r_{ij}(t)) \cdot (\Delta x_{i}(t) - \Delta x_{j}(t)) $

$ = m (\Delta x_{i}(t) \sum _{j}\nabla W_{ij} $

$ - \sum_{j}\nabla W_{ij}\Delta x_{j}(t)) $

因为 $ \Delta x_{i} = \Delta t ^{2} \frac{F_{i}^{p}}{m}​$

而压力 $f_{p}= m^{2}  \sum_{j} (\frac{p_{i}}{\rho _{i} ^{2}}$
$ + \frac{p_{j}}{\rho _{j} ^{2}}) \nabla W(|x - x_{j}|)​$,
可得 $F_{j =i}^p = m^2 \sum _{j} \frac{p_{i}$
$ + p_{i}}{\rho_{0}^{2}}\nabla W_{ij}​$

代入一下可得$\Delta x_{i} = - \Delta t^{2} m \frac{2p_{i}}{\rho_{0}^2}\sum _{j} \nabla W_{ij}​$，$\Delta x_{j} = - \Delta t^{2} m \frac{2p_{i}}{\rho_{0}^2}\nabla W_{ij}​$

把上式代入密度增量公式，可得 $\Delta \rho_{i}(t) =\Delta t^{2} m^{2} \frac{2p_{i}}{\rho_{0}^2}(-\sum_{j}\nabla W_{ij} \cdot \sum_{j}\nabla W_{ij} - \sum_{j}(\nabla W_{ij} \cdot \nabla W_{ij})) $

把上式转换一下，可得压强计算公式 $p_{i} = \frac{\Delta \rho_{i}(t) }{(-\sum_{j}\nabla W_{ij} \cdot \sum_{j}\nabla W_{ij} - \sum_{j}(\nabla W_{ij} \cdot \nabla W_{ij})\beta }​$  

这串公式可以这么理解，当希望增量密度 $\Delta \rho _{i}(t)​$,需要施加压强pi 

其中 $\beta =\Delta t^{2} m^{2} \frac{2}{\rho_{0}^2}​$

设当前密度为 
$\rho_{i}^{*}​$ 
目标密度为 
$\rho_{0}​$ 
则
$\Delta \rho_{i}(t) = \rho_{0} - \rho_{i}^{*} = - \rho_{error}​$

设 $p_{i} = \delta \rho_{error}​$ 可得 $\delta = \frac{-1}{(-\sum_{j}\nabla W_{ij} \cdot \sum_{j}\nabla W_{ij} - \sum_{j}(\nabla W_{ij} \cdot \nabla W_{ij}))\beta }​$

$p_{i}(t) += \delta \rho_{error}$

通过压强更新公式，我们可以不断的在矫正粒子密度使其接近不可压缩条件

代码实现如下

```c++
double CalfFluidEngine::PCISPHSolver3::computeDelta(double timeStepInSeconds)
{
	auto particles = GetSphData();
	const double kernelRadius = particles->GetKernelRadius();

	std::vector<Vector3D> points;
	BccLatticePointGenerator pointsGenerator;
	Vector3D origin = Vector3D::zero;
	BoundingBox3D sampleBound(origin, origin);
	sampleBound.Expand(1.5 * kernelRadius);

	pointsGenerator.Generate(sampleBound, particles->GetTargetSpacing(), &points);

	SphSpikyKernel3 kernel(kernelRadius);

	double denom = 0;
	Vector3D denom1 = Vector3D::zero;
	double denom2 = 0;

	for (size_t i = 0; i < points.size(); ++i) {
		const Vector3D& point = points[i];
		double distanceSquared = point.SquareMagnitude();

		if (distanceSquared < kernelRadius * kernelRadius) {
			double distance = std::sqrt(distanceSquared);
			Vector3D direction =
				(distance > 0.0) ? point / distance : Vector3D::zero;

			// grad(Wij)
			Vector3D gradWij = kernel.Gradient(distance, direction);
			denom1 += gradWij;
			denom2 += Vector3D::Dot(gradWij,gradWij);
		}
	}

	denom += -Vector3D::Dot(denom1,denom1) - denom2;

	return (std::fabs(denom) > 0.0) ?
		-1 / (computeBeta(timeStepInSeconds) * denom) : 0;
}

double CalfFluidEngine::PCISPHSolver3::computeBeta(double timeStepInSeconds)
{
	auto particles = GetSphData();
	double t = particles->GetParticleMass() * timeStepInSeconds
		/ particles->GetDensity();
	return 2.0 * t * t;
}

```

这边再一次列出计算密度误差的代码



```c++
// Compute pressure from density error
		tbb::parallel_for(
			tbb::blocked_range<size_t>(0, numberOfParticles),
			[&](const tbb::blocked_range<size_t> & b) {
			for (size_t i = b.begin(); i != b.end(); ++i)
			{
				double weightSum = 0.0;
				const auto& neighbors = particles->GetNeighborLists()[i];

				for (size_t j : neighbors) {
					double dist = Vector3D::Distance(_tempPositions[j], _tempPositions[i]);
					weightSum += kernel(dist);
				}
	
				weightSum += kernel(0);

				double density = mass * weightSum;
				double densityError = (density - targetDensity);
				double pressure = delta * densityError;

				if (pressure < 0.0) {
					pressure *= _negativePressureScale;
					densityError *= _negativePressureScale;
				}

				p[i] += pressure;
				predictedDensities[i] = density;
				_densityErrors[i] = densityError;
			}
		});
```

