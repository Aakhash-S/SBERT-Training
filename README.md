describe('processComponentDependencies', () => {
  
  it('should set allTrue to true if keys[0] is "OR" and at least one dependencyValidationList item is true', () => {
    const taskStep = { components: [] };
    const compConfig = {
      compDependencyModel: [{ type: 'SHOWHIDE' }],
      id: 1
    };

    spyOn(service.utils, 'isEmpty').and.returnValue(false);

    const dependencyValidationList = [false, true, false];
    spyOn(service, 'processDataMap').and.returnValue(dependencyValidationList);

    const result = service.processComponentDependencies(taskStep, compConfig);

    expect(result).toBeTruthy();
  });

  it('should set allTrue to true only if all dependencyValidationList values are true and keys[0] is not "OR"', () => {
    const taskStep = { components: [] };
    const compConfig = {
      compDependencyModel: [{ type: 'SHOWHIDE' }],
      id: 1
    };

    spyOn(service.utils, 'isEmpty').and.returnValue(false);

    const dependencyValidationList = [true, true, true];
    spyOn(service, 'processDataMap').and.returnValue(dependencyValidationList);

    const result = service.processComponentDependencies(taskStep, compConfig);

    expect(result).toBeTruthy();
  });

  it('should set targetComponent.isVisible to true when allTrue is true for SHOWHIDE type', () => {
    const targetComponent = { isVisible: false };
    const taskStep = { components: [targetComponent] };
    const compConfig = {
      compDependencyModel: [{ type: 'SHOWHIDE' }],
      id: 1
    };

    spyOn(service.utils, 'isEmpty').and.returnValue(false);

    const dependencyValidationList = [true];
    spyOn(service, 'processDataMap').and.returnValue(dependencyValidationList);

    service.processComponentDependencies(taskStep, compConfig);

    expect(targetComponent.isVisible).toBeTrue();
  });

  it('should clear targetComponent.selected for DecisionCheckbox type', () => {
    const targetComponent = { type: 'DecisionCheckbox', selected: [1, 2, 3] };
    const taskStep = { components: [targetComponent] };
    const compConfig = {
      compDependencyModel: [{ type: 'SHOWHIDE' }],
      id: 1
    };

    spyOn(service.utils, 'isEmpty').and.returnValue(false);

    service.processComponentDependencies(taskStep, compConfig);

    expect(targetComponent.selected).toEqual([]);
  });

  it('should set targetComponent.readOnly to allTrue for DecisionRadio or DecisionYesNo types', () => {
    const targetComponent = { type: 'DecisionRadio', readOnly: false };
    const taskStep = { components: [targetComponent] };
    const compConfig = {
      compDependencyModel: [{ type: 'READEDIT' }],
      id: 1
    };

    spyOn(service.utils, 'isEmpty').and.returnValue(false);

    const dependencyValidationList = [true];
    spyOn(service, 'processDataMap').and.returnValue(dependencyValidationList);

    service.processComponentDependencies(taskStep, compConfig);

    expect(targetComponent.readOnly).toBeTrue();
  });

});
