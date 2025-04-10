describe('removeCurrentStep', () => {
  let service: CollateralTaskStepsMapperService;
  let mockStepConfig: any;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [CollateralTaskStepsMapperService],
    });

    service = TestBed.inject(CollateralTaskStepsMapperService);

    service.guidedTaskModel = {
      steps: [
        { stepId: 1 },
        { stepId: 2 },
        { stepId: 3 },
      ]
    };

    service.taskProfileConfig = {
      taskList: [
        { stepId: 1 },
        { stepId: 2 },
        { stepId: 3 },
      ]
    };

    service.taskValidationService = {
      dataSetters: jasmine.createSpy('dataSetters')
    };

    service.taskModel = {
      guidedTaskModel: {}
    };

    mockStepConfig = { stepId: 2 };
  });

  it('should remove the current step and update stepIds correctly', () => {
    service.removeCurrentStep(mockStepConfig);

    // Expect step with stepId: 2 to be removed
    expect(service.guidedTaskModel.steps.length).toBe(2);
    expect(service.guidedTaskModel.steps.find(s => s.stepId === 2)).toBeUndefined();

    // StepIds should be updated
    expect(service.guidedTaskModel.steps[0].stepId).toBe(1);
    expect(service.guidedTaskModel.steps[1].stepId).toBe(2);

    // taskProfileConfig taskList stepIds should also be updated
    expect(service.taskProfileConfig.taskList[0].stepId).toBe(1);
    expect(service.taskProfileConfig.taskList[1].stepId).toBe(2);

    // Should call dataSetters
    expect(service.taskValidationService.dataSetters).toHaveBeenCalledWith(
      service.guidedTaskModel,
      service.taskProfileConfig,
      service.taskInfoGetResponse
    );

    // taskModel.guidedTaskModel should be updated
    expect(service.taskModel.guidedTaskModel.steps.length).toBe(2);
  });

  it('should not fail if guidedTaskModel.steps is empty', () => {
    service.guidedTaskModel.steps = [];
    service.removeCurrentStep(mockStepConfig);

    expect(service.guidedTaskModel.steps.length).toBe(0);
    expect(service.taskValidationService.dataSetters).toHaveBeenCalled();
    expect(service.taskModel.guidedTaskModel.steps).toEqual([]);
  });
});
