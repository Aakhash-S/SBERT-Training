describe('removeCurrentStep', () => {
  let serviceUnderTest: YourServiceClass; // Replace with your actual class
  let mockTaskValidationService: any;

  beforeEach(() => {
    mockTaskValidationService = {
      dataSetters: jasmine.createSpy()
    };

    serviceUnderTest = new YourServiceClass(/* inject dependencies as needed */);
    serviceUnderTest['taskValidationService'] = mockTaskValidationService;

    serviceUnderTest['taskProfileConfig'] = {
      taskList: [
        { stepId: 0 },
        { stepId: 1 },
        { stepId: 2 }
      ]
    };

    serviceUnderTest['guidedTaskModel'] = {
      steps: [
        { stepId: 0 },
        { stepId: 1 },
        { stepId: 2 }
      ]
    };

    serviceUnderTest['taskModel'] = {
      guidedTaskModel: {}
    };
  });

  it('should remove the step and update stepIds', () => {
    const stepConfig = { stepId: 2 };

    serviceUnderTest.removeCurrentStep(stepConfig);

    expect(serviceUnderTest['guidedTaskModel'].steps.length).toBe(2);
    expect(serviceUnderTest['taskProfileConfig'].taskList.length).toBe(2);

    expect(serviceUnderTest['guidedTaskModel'].steps[0].stepId).toBe(0);
    expect(serviceUnderTest['guidedTaskModel'].steps[1].stepId).toBe(1);

    expect(serviceUnderTest['taskProfileConfig'].taskList[0].stepId).toBe(0);
    expect(serviceUnderTest['taskProfileConfig'].taskList[1].stepId).toBe(1);

    expect(mockTaskValidationService.dataSetters).toHaveBeenCalledWith(
      serviceUnderTest['guidedTaskModel'],
      serviceUnderTest['taskProfileConfig'],
      serviceUnderTest['taskInfoGetResponse']
    );

    expect(serviceUnderTest['taskModel'].guidedTaskModel).toEqual(
      jasmine.objectContaining(serviceUnderTest['guidedTaskModel'])
    );
  });
});

describe('triggerBrowserEventsForDisposition', () => {
  let serviceUnderTest: YourServiceClass; // Replace with actual class name
  let mockTaskStepService: any;
  let mockRouter: any;
  let mockLoggerService: any;
  let mockApprovalLayerService: any;
  let mockCollateralTaskStepsMapperService: any;

  beforeEach(() => {
    mockTaskStepService = { dispositionData: jasmine.createSpy().and.returnValue(of({})) };
    mockRouter = { navigate: jasmine.createSpy() };
    mockLoggerService = { log: jasmine.createSpy() };
    mockApprovalLayerService = { hideSpinner: jasmine.createSpy() };
    mockCollateralTaskStepsMapperService = {
      updateSubTaskForBrowserEvents: jasmine.createSpy(),
      getCurrentTask: jasmine.createSpy().and.returnValue({ subTasks: ['mockSubTask1'] })
    };

    serviceUnderTest = new YourServiceClass( // Replace with constructor and args as needed
      mockTaskStepService,
      mockRouter,
      mockLoggerService,
      mockApprovalLayerService,
      mockCollateralTaskStepsMapperService
    );

    serviceUnderTest['taskProfileConfig'] = mockDocumentValidstionTaskProfile;
    serviceUnderTest['contextService'] = {
      getUserContext: () => ({ userName: 'mockUser' })
    };
  });

  it('should call dispositionData and navigate on success', (done) => {
    serviceUnderTest.triggerBrowserEventsForDisposition().then(() => {
      expect(mockTaskStepService.dispositionData).toHaveBeenCalled();
      expect(mockRouter.navigate).toHaveBeenCalledWith(['/nfe-collateral-task-web', 'home']);
      expect(mockApprovalLayerService.hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should navigate to 500 and hide spinner on error', (done) => {
    mockTaskStepService.dispositionData.and.returnValue(throwError(() => new Error('Mock error')));
    spyOn(console, 'error');

    serviceUnderTest.triggerBrowserEventsForDisposition().catch(() => {
      expect(console.error).toHaveBeenCalled();
      expect(mockRouter.navigate).toHaveBeenCalledWith(['/500']);
      expect(mockApprovalLayerService.hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should resolve immediately when profileInfo is not available', (done) => {
    serviceUnderTest['taskProfileConfig'].profileInfo = null as any;
    spyOn(serviceUnderTest, 'hideLeftNavBar');

    serviceUnderTest.triggerBrowserEventsForDisposition().then(() => {
      expect(serviceUnderTest.hideLeftNavBar).toHaveBeenCalledWith(false);
      done();
    });
  });
});
