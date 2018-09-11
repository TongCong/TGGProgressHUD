# TGGProgressHUD

/** */
+ (void)showStatus:(NSString *)status;
/** */
+ (void)showStatus:(NSString *)status andAutoHideAfterTime:(CGFloa t)showTim e;
/** */
+ (void)showSuccess;
/** */
+ (void)showSuccessAndAutoHideAfterTime:(CGFloat)showTime;
/** */
+ (void)showSuccessWithStatus:(NSString *)status;
/** */
+ (void)showSuccessWithStatus:(NSString *)status andAutoHideAfterTi me:(CGFlo at)showTime;
/** */
+ (void)showError;
/** */
+ (void)showErrorAndAutoHideAfterTime:(CGFloat)showTime;
/** */
+ (void)showErrorWithStatus:(NSString *)status;
/** */
+ (void)showErrorWithStatus:(NSString *)status andAutoHideAfterTim e:(CGFloa t)showTime;
/** */
+ (void)showProgress;
/**
d)showProgressWithStatus:(NSString *)status;
/** */
+ (void)showCustomHUD:(UIView *)hudView;
/** */
+ (void)showCustomHUD:(UIView *)hudView andAutoHideAfterTime:(CGFlo at)showTi me;
/** */
+ (void)hideHUD;
/** */
+ (void)hideAllHUDs; @end
