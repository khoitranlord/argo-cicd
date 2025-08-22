# Infrastructure Deployment Request

## 📋 Deployment Details

**Application**: 
**Environment**: [ ] dev [ ] staging [ ] prod
**Image/Version**: 
**Expected Impact**: [ ] Low [ ] Medium [ ] High

## 🔍 Pre-deployment Checklist

### Security & Compliance
- [ ] Security scan completed and passed
- [ ] No sensitive data in configurations
- [ ] RBAC permissions reviewed
- [ ] Network policies validated

### Testing & Quality
- [ ] Application tests passed
- [ ] Integration tests completed
- [ ] Performance benchmarks met
- [ ] Smoke tests prepared

### Operational Readiness
- [ ] Monitoring and alerting configured
- [ ] Logging properly set up
- [ ] Health checks defined
- [ ] Rollback procedure documented

## 📊 Impact Assessment

### Resource Requirements
- **CPU**: 
- **Memory**: 
- **Storage**: 
- **Network**: 

### Dependencies
- [ ] Database migrations reviewed
- [ ] External service compatibility checked
- [ ] Configuration dependencies verified

## 🚨 Risk Assessment

**Risk Level**: [ ] 🟢 Low [ ] 🟡 Medium [ ] 🔴 High

**Mitigation Strategies**:
- 
- 
- 

## 🔄 Post-deployment Verification

### Immediate Checks (0-15 minutes)
- [ ] Pods are running and healthy
- [ ] Service endpoints responding
- [ ] No error spikes in logs

### Extended Monitoring (15-60 minutes)
- [ ] Performance metrics stable
- [ ] Error rates within acceptable limits
- [ ] User experience unaffected

### Success Criteria
- [ ] All health checks pass
- [ ] Response times < XXXms
- [ ] Error rate < X%

## 👥 Required Approvals

- [ ] **Platform Team** (@platform-team) - Always required
- [ ] **SRE Team** (@sre-team) - Required for staging/prod
- [ ] **Security Team** (@security-team) - Required for security changes
- [ ] **Product Owner** - Required for feature changes

## 📝 Additional Notes

<!-- Add any specific notes, concerns, or context here -->

---

**⚠️ Important**: This deployment will be automatically applied by ArgoCD once merged. Ensure all checks are completed before approval.