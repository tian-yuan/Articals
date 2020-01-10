### Fiber 原理及使用

#### 使用

```
TEST(FiberManager, batonTimedWaitTimeout) {
  bool taskAdded = false;
  size_t iterations = 0;

  FiberManager manager(std::make_unique<SimpleLoopController>());
  auto& loopController =
      dynamic_cast<SimpleLoopController&>(manager.loopController());

  auto now = SimpleLoopController::Clock::now();
  loopController.setTimeFunc([&] { return now; });

  auto loopFunc = [&]() {
    if (!taskAdded) {
      manager.addTask([&]() {
        Baton baton;

        auto res = baton.try_wait_for(std::chrono::milliseconds(230));

        EXPECT_FALSE(res);
        EXPECT_EQ(5, iterations);

        loopController.stop();
      });
      manager.addTask([&]() {
        Baton baton;

        auto res = baton.try_wait_for(std::chrono::milliseconds(130));

        EXPECT_FALSE(res);
        EXPECT_EQ(3, iterations);

        loopController.stop();
      });
      taskAdded = true;
    } else {
      now += std::chrono::milliseconds(50);
      iterations++;
    }
  };

  loopController.loop(std::move(loopFunc));
}
```

baton.try_wait_for 在等待时，会切换到其他 task 运行，在 Baton-inl.h 中可以看到会获取 FiberManager，激活其他 task 运行，所以一个协程可以并行地运行多个任务。