describe('triggerBrowserEventsForDisposition', () => {
  let service: CollateralTaskStepsMapperService;
  let router: Router;
  let taskStepService: any;
  let loggerService: any;
  let approvalLayerService: any;
  let appStateMgmt: any;

  beforeEach(() => {
    service = TestBed.inject(CollateralTaskStepsMapperService);
    router = TestBed.inject(Router);
    taskStepService = TestBed.inject(CollateralTaskStepService);
    loggerService = TestBed.inject(LoggerService);
    approvalLayerService = TestBed.inject(ApprovalLayerService);
    appStateMgmt = TestBed.inject(CollateralTaskMapperService);

    service.taskProfileConfig = {
      profileInfo: {
        taskId: '123',
        taskName: 'Some Task',
        userId: 'user123'
      }
    };

    service.contextService = {
      getUserContext: () => ({ userName: 'testUser' })
    };

    spyOn(appStateMgmt, 'getCurrentTask').and.returnValue({ subTasks: ['sub1'] });
  });

  it('should call dispositionData and navigate on success', (done) => {
    const expectedRequest = {
      taskId: '123',
      taskName: 'Some Task',
      userId: 'user123',
      subTasks: ['sub1']
    };

    spyOn(taskStepService, 'dispositionData').and.returnValue(of({}));
    spyOn(approvalLayerService, 'hideSpinner');
    spyOn(router, 'navigate');

    service.triggerBrowserEventsForDisposition().then(() => {
      expect(taskStepService.dispositionData).toHaveBeenCalledWith(expectedRequest);
      expect(router.navigate).toHaveBeenCalledWith(['/nfe-collateral-task-web', 'home']);
      expect(approvalLayerService.hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should navigate to 500 and hide spinner on error', (done) => {
    spyOn(taskStepService, 'dispositionData').and.returnValue(throwError(() => new Error('Error')));
    spyOn(router, 'navigate');
    spyOn(approvalLayerService, 'hideSpinner');
    spyOn(console, 'error');

    service.triggerBrowserEventsForDisposition().catch(() => {
      expect(console.error).toHaveBeenCalled();
      expect(router.navigate).toHaveBeenCalledWith(['/500']);
      expect(approvalLayerService.hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should resolve immediately when profileInfo is not available', (done) => {
    service.taskProfileConfig.profileInfo = null as any;
    spyOn(service, 'hideLeftNavBar');

    service.triggerBrowserEventsForDisposition().then(() => {
      expect(service.hideLeftNavBar).toHaveBeenCalledWith(false);
      done();
    });
  });
});
