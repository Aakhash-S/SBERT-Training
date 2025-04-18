describe('triggerBrowserEventsForDisposition', () => {
  beforeEach(() => {
    // Setup mocks
    spyOn(console, 'error');
    
    // Override required service methods
    service.taskProfileConfig = {
      profileInfo: {
        taskId: '123',
        taskName: 'Some Task',
        userId: 'user123'
      }
    };

    service.contextService = {
      getUserContext: () => ({ userName: 'mockUser' })
    };

    // Setup the mock to return success response
    const mockTaskStepService = TestBed.inject(CollateralTaskStepService) as any;
    mockTaskStepService.dispositionData = jasmine.createSpy().and.returnValue(of({}));

    const mockAppStateMgmtService = TestBed.inject(CollateralTaskStepsMapperService) as any;
    mockAppStateMgmtService.getCurrentTask = jasmine.createSpy().and.returnValue({ subTasks: ['mockSubTask1'] });
    mockAppStateMgmtService.updateSubtaskForBrowserEvents = jasmine.createSpy();
  });

  it('should call dispositionData and navigate on success', (done) => {
    const mockRouter = TestBed.inject(Router) as any;
    const mockApprovalLayerService = TestBed.inject(CollateralTaskStepsValidationService) as any;

    service.triggerBrowserEventsForDisposition().then(() => {
      expect(TestBed.inject(CollateralTaskStepService).dispositionData).toHaveBeenCalled();
      expect(mockRouter.navigate).toHaveBeenCalledWith(['/nfe-collateral-task-web', 'home']);
      expect(mockApprovalLayerService.hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should navigate to 500 and hide spinner on error', (done) => {
    const mockTaskStepService = TestBed.inject(CollateralTaskStepService) as any;
    mockTaskStepService.dispositionData.and.returnValue(throwError(() => new Error('Mock error')));

    const mockRouter = TestBed.inject(Router) as any;
    const mockApprovalLayerService = TestBed.inject(CollateralTaskStepsValidationService) as any;

    service.triggerBrowserEventsForDisposition().catch(() => {
      expect(console.error).toHaveBeenCalled();
      expect(mockRouter.navigate).toHaveBeenCalledWith(['/500']);
      expect(mockApprovalLayerService.hideSpinner).toHaveBeenCalled();
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
