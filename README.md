describe('getTaskStep', () => {
  
  it('should return the first filtered step when guidedTaskModel.steps is not empty', () => {
    const dependencyStep = { stepId: 1 };
    service.guidedTaskModel = {
      steps: [{ stepId: 1, name: 'Step 1' }, { stepId: 2, name: 'Step 2' }]
    };

    const result = service.getTaskStep(dependencyStep);

    expect(result).toEqual({ stepId: 1, name: 'Step 1' });
  });

  it('should return undefined when guidedTaskModel.steps is empty', () => {
    const dependencyStep = { stepId: 3 };
    service.guidedTaskModel = { steps: [] };

    const result = service.getTaskStep(dependencyStep);

    expect(result).toBeUndefined();
  });

});

describe('getTaskStepComponent', () => {
  
  it('should return the first filtered component when taskStep has matching components', () => {
    const dependencyStep = { stepId: 1, componentId: 101 };
    
    spyOn(service, 'getTaskStep').and.returnValue({
      stepId: 1,
      components: [{ id: 101, name: 'Component 1' }, { id: 102, name: 'Component 2' }]
    });

    const result = service.getTaskStepComponent(dependencyStep);

    expect(result).toEqual({ id: 101, name: 'Component 1' });
  });

  it('should return undefined when taskStep has no matching components', () => {
    const dependencyStep = { stepId: 2, componentId: 201 };
    
    spyOn(service, 'getTaskStep').and.returnValue({
      stepId: 2,
      components: [{ id: 202, name: 'Component A' }]
    });

    const result = service.getTaskStepComponent(dependencyStep);

    expect(result).toBeUndefined();
  });

  it('should return undefined when taskStep is empty', () => {
    const dependencyStep = { stepId: 3, componentId: 301 };
    
    spyOn(service, 'getTaskStep').and.returnValue(undefined);

    const result = service.getTaskStepComponent(dependencyStep);

    expect(result).toBeUndefined();
  });

});
