# Flutter raw multi-view

A prototype that renders to multiple views.

You can [modify the Flutter engine](https://github.com/flutter/flutter/wiki/Compiling-the-engine)
to create multiple views back by the same surface:

```diff
diff --git a/lib/ui/window/platform_configuration.cc b/lib/ui/window/platform_configuration.cc
index 4e6e583745..043c191ad3 100644
--- a/lib/ui/window/platform_configuration.cc
+++ b/lib/ui/window/platform_configuration.cc
@@ -76,6 +76,12 @@ void PlatformConfiguration::DidCreateIsolate() {
   windows_.emplace(kImplicitViewId,
                    std::make_unique<Window>(
                        kImplicitViewId, ViewportMetrics{1.0, 0.0, 0.0, -1}));
+
+  // HACK HACK. Add a second view with ID 1 and a third view with ID 2.
+  windows_.emplace(
+      1, std::make_unique<Window>(1, ViewportMetrics{1.0, 0.0, 0.0, -1}));
+  windows_.emplace(
+      2, std::make_unique<Window>(2, ViewportMetrics{1.0, 0.0, 0.0, -1}));
 }
 
 void PlatformConfiguration::UpdateLocales(
diff --git a/runtime/runtime_controller.cc b/runtime/runtime_controller.cc
index 996cc38114..19e2223c07 100644
--- a/runtime/runtime_controller.cc
+++ b/runtime/runtime_controller.cc
@@ -127,6 +127,8 @@ bool RuntimeController::SetViewportMetrics(const ViewportMetrics& metrics) {
 
   if (auto* platform_configuration = GetPlatformConfigurationIfAvailable()) {
     platform_configuration->get_window(0)->UpdateWindowMetrics(metrics);
+    // HACK HACK. Also send view metrics for second and third view.
+    platform_configuration->get_window(1)->UpdateWindowMetrics(metrics);
+    platform_configuration->get_window(2)->UpdateWindowMetrics(metrics);
     return true;
   }
 
diff --git a/shell/common/pipeline.h b/shell/common/pipeline.h
index 1de4498e7f..a4abc83548 100644
--- a/shell/common/pipeline.h
+++ b/shell/common/pipeline.h
@@ -100,6 +100,8 @@ class Pipeline {
         continuation_ = nullptr;
         TRACE_EVENT_ASYNC_END0("flutter", "PipelineProduce", trace_id_);
         TRACE_FLOW_STEP("flutter", "PipelineItem", trace_id_);
+      } else {
+        // The engine assumes a single render per frame currently.
+        FML_LOG(ERROR) << "Continuation is null!";
       }
       return result;
     }
```