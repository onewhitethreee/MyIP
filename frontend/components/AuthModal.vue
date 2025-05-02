<template>
    <div class="modal fade" id="authModal" tabindex="-1" aria-labelledby="authModalLabel" aria-hidden="true"
        data-bs-backdrop="static">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content border-0 shadow-lg">
                <div class="modal-header border-0 pb-0">
                    <h5 class="modal-title fw-bold">
                        {{ currentForm === 'login' ? t('auth.login') : currentForm === 'register' ? t('auth.register') :
                            t('auth.forgotPassword') }}
                    </h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"
                        @click="closeModal"></button>
                </div>
                <div class="modal-body p-4">
                    <div class="auth-forms-container">
                        <!-- Removed transition component -->
                        <form v-if="currentForm === 'login'" @submit.prevent="handleLogin" class="auth-form"
                            key="login">
                            <div class="mb-4">
                                <label for="loginEmail" class="form-label fw-medium">{{ t('auth.email') }}</label>
                                <div class="input-group">
                                    <span class="input-group-text bg-light border-end-0">
                                        <i class="bi bi-envelope"></i>
                                    </span>
                                    <input type="email" class="form-control border-start-0 ps-0" id="loginEmail"
                                        v-model="loginForm.email" required placeholder="your@email.com">
                                </div>
                            </div>
                            <div class="mb-4">
                                <div class="d-flex justify-content-between align-items-center">
                                    <label for="loginPassword" class="form-label fw-medium mb-0">{{
                                        t('auth.password') }}</label>
                                    <a href="#" class="text-decoration-none small text-primary"
                                        @click.prevent="switchForm('forgot')">
                                        {{ t('auth.forgotPassword') }}
                                    </a>
                                </div>
                                <div class="input-group mt-2">
                                    <span class="input-group-text bg-light border-end-0">
                                        <i class="bi bi-lock"></i>
                                    </span>
                                    <input :type="showPassword ? 'text' : 'password'"
                                        class="form-control border-start-0 border-end-0 ps-0" id="loginPassword"
                                        v-model="loginForm.password" required placeholder="••••••••" autocomplete="current-password">
                                    <span class="input-group-text bg-light border-start-0 cursor-pointer"
                                        @click="togglePassword">
                                        <i :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"></i>
                                    </span>
                                </div>
                            </div>
                            <button type="submit" class="btn btn-primary w-100 py-2 fw-medium shadow-sm">
                                {{ t('auth.login') }}
                            </button>

                            <div class="mt-4 text-center">
                                <div class="divider">
                                    <span class="divider-text">{{ t('auth.orLoginWith') }}</span>
                                </div>

                                <div class="social-login mt-3">
                                    <button type="button" class="btn btn-social btn-google" disabled>
                                        <i class="bi bi-google"></i>
                                    </button>
                                    <button type="button" class="btn btn-social btn-github" disabled>
                                        <i class="bi bi-github"></i>
                                    </button>
                                    <button type="button" class="btn btn-social btn-facebook" disabled>
                                        <i class="bi bi-facebook"></i>
                                    </button>
                                </div>

                                <div class="mt-4">
                                    <span class="text-muted">{{ t('auth.noAccount') }}</span>
                                    <a href="#" class="text-decoration-none ms-1 fw-medium"
                                        @click.prevent="switchForm('register')">
                                        {{ t('auth.register') }}
                                    </a>
                                </div>
                            </div>
                        </form>

                        <form v-else-if="currentForm === 'register'" @submit.prevent="handleRegister" class="auth-form"
                            key="register">
                            
                            <div class="mb-3">
                                <label for="registerEmail" class="form-label fw-medium">{{ t('auth.email')
                                }}</label>
                                <div class="input-group">
                                    <span class="input-group-text bg-light border-end-0">
                                        <i class="bi bi-envelope"></i>
                                    </span>
                                    <input type="email" class="form-control border-start-0 ps-0" id="registerEmail"
                                        v-model="registerForm.email" required placeholder="your@email.com">
                                </div>
                            </div>
                            <div class="mb-3">
                                <label for="registerPassword" class="form-label fw-medium">{{ t('auth.password')
                                }}</label>
                                <div class="input-group">
                                    <span class="input-group-text bg-light border-end-0">
                                        <i class="bi bi-lock"></i>
                                    </span>
                                    <input :type="showPassword ? 'text' : 'password'"
                                        class="form-control border-start-0 border-end-0 ps-0" id="registerPassword"
                                        v-model="registerForm.password" required placeholder="••••••••">
                                    <span class="input-group-text bg-light border-start-0 cursor-pointer"
                                        @click="togglePassword">
                                        <i :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"></i>
                                    </span>
                                </div>
                            </div>
                            <div class="mb-4">
                                <label for="registerPasswordConfirm" class="form-label fw-medium">{{
                                    t('auth.confirmPassword') }}</label>
                                <div class="input-group">
                                    <span class="input-group-text bg-light border-end-0">
                                        <i class="bi bi-lock-fill"></i>
                                    </span>
                                    <input :type="showPassword ? 'text' : 'password'"
                                        class="form-control border-start-0 border-end-0 ps-0"
                                        id="registerPasswordConfirm" v-model="registerForm.passwordConfirm" required
                                        placeholder="••••••••">
                                    <span class="input-group-text bg-light border-start-0 cursor-pointer"
                                        @click="togglePassword">
                                        <i :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"></i>
                                    </span>
                                </div>
                            </div>
                            <button type="submit" class="btn btn-primary w-100 py-2 fw-medium shadow-sm">
                                {{ t('auth.register') }}
                            </button>

                            <div class="mt-4 text-center">
                                <span class="text-muted">{{ t('auth.alreadyHaveAccount') }}</span>
                                <a href="#" class="text-decoration-none ms-1 fw-medium"
                                    @click.prevent="switchForm('login')">
                                    {{ t('auth.backToLogin') }}
                                </a>
                            </div>
                        </form>

                        <form v-else-if="currentForm === 'forgot'" @submit.prevent="handleForgotPassword"
                            class="auth-form" key="forgot">
                            <div class="text-center mb-4">
                                <div class="reset-icon-container mb-3">
                                    <i class="bi bi-envelope-paper fs-1 text-primary"></i>
                                </div>
                                <p class="text-muted">{{ t('auth.forgotPasswordInstructions') || 'Enter your email address and we will send you instructions to reset your password.' }}</p>
                            </div>
                            <div class="mb-4">
                                <label for="forgotEmail" class="form-label fw-medium">{{ t('auth.email') }}</label>
                                <div class="input-group">
                                    <span class="input-group-text bg-light border-end-0">
                                        <i class="bi bi-envelope"></i>
                                    </span>
                                    <input type="email" class="form-control border-start-0 ps-0" id="forgotEmail"
                                        v-model="forgotForm.email" required placeholder="your@email.com">
                                </div>
                            </div>
                            <button type="submit" class="btn btn-primary w-100 py-2 fw-medium shadow-sm">
                                {{ t('auth.resetPassword') }}
                            </button>

                            <div class="mt-4 text-center">
                                <a href="#" class="text-decoration-none fw-medium" @click.prevent="switchForm('login')">
                                    <i class="bi bi-arrow-left me-1"></i> {{ t('auth.backToLogin') }}
                                </a>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import { Modal } from 'bootstrap';

const props = defineProps({
    show: {
        type: Boolean,
        required: true
    }
});

const emit = defineEmits(['close']);

const { t } = useI18n();

// 表单状态
const currentForm = ref('login');
// Removed previousForm and transitionName variables as they were used for animations
const showPassword = ref(false);

// 表单数据
const loginForm = ref({
    email: '',
    password: ''
});

const registerForm = ref({
    name: '',
    email: '',
    password: '',
    passwordConfirm: ''
});

const forgotForm = ref({
    email: ''
});

// Removed formFlow object as it was used for animations

// Modal 实例
const modal = ref(null);
const isModalInitialized = ref(false);

const initModal = () => {
    const modalElement = document.getElementById('authModal');
    if (modalElement) {
        modal.value = new Modal(modalElement, {
            backdrop: 'static',
            keyboard: false
        });
        isModalInitialized.value = true;
        if (props.show) {
            modal.value.show();
        }
    }
};

onMounted(() => {
    initModal();
});

onUnmounted(() => {
    if (modal.value) {
        modal.value.hide();
        modal.value.dispose();
    }
    isModalInitialized.value = false;
});

watch(() => props.show, (newValue) => {
    if (isModalInitialized.value) {
        if (newValue) {
            modal.value.show();
        } else {
            modal.value.hide();
        }
    } else if (newValue) {
        initModal();
    }
});

// 切换密码可见性
const togglePassword = () => {
    showPassword.value = !showPassword.value;
};

// 表单处理函数
const handleLogin = () => {
    // TODO: Implement login logic
    console.log('Login attempt:', loginForm.value);
};

const handleRegister = () => {
    // TODO: Implement register logic
    console.log('Register attempt:', registerForm.value);
};

const handleForgotPassword = () => {
    // TODO: Implement forgot password logic
    console.log('Forgot password attempt:', forgotForm.value);
};

// 切换表单 - simplified without animation logic
const switchForm = (form) => {
    currentForm.value = form;
};

// 关闭模态框
const closeModal = () => {
    if (modal.value) {
        modal.value.hide();
    }
    emit('close');
};
</script>

<style scoped>
.modal-content {
    border-radius: 12px;
    overflow: hidden;
}

.modal-header {
    padding: 1.5rem 1.5rem 0.5rem;
}

.modal-title {
    font-size: 1.5rem;
    font-weight: 600;
    color: #333;
}

.auth-forms-container {
    position: relative;
    min-height: 300px;
}

.auth-form {
    width: 100%;
}

/* Removed all animation-related CSS classes */

.form-label {
    color: #555;
    font-size: 0.9rem;
}

.input-group-text {
    color: #6c757d;
}

.form-control {
    padding: 0.6rem 0.75rem;
    background-color: #f8f9fa;
}

.form-control:focus {
    background-color: #fff;
    box-shadow: 0 0 0 0.25rem rgba(13, 110, 253, 0.15);
}

.btn-primary {
    background-color: #0d6efd;
    border-color: #0d6efd;
    transition: all 0.2s ease;
}

.btn-primary:hover {
    background-color: #0b5ed7;
    border-color: #0a58ca;
    transform: translateY(-1px);
}

.btn-primary:active {
    transform: translateY(0);
}

.divider {
    display: flex;
    align-items: center;
    text-align: center;
    margin: 1rem 0;
}

.divider::before,
.divider::after {
    content: '';
    flex: 1;
    border-bottom: 1px solid #dee2e6;
}

.divider-text {
    padding: 0 1rem;
    color: #6c757d;
    font-size: 0.85rem;
}

.social-login {
    display: flex;
    justify-content: center;
    gap: 12px;
}

.btn-social {
    width: 42px;
    height: 42px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s ease;
    color: #fff;
    border: none;
}

.btn-social:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.btn-google {
    background-color: #ea4335;
}

.btn-github {
    background-color: #333;
}

.btn-facebook {
    background-color: #3b5998;
}

.cursor-pointer {
    cursor: pointer;
}

.reset-icon-container {
    width: 70px;
    height: 70px;
    border-radius: 50%;
    background-color: rgba(13, 110, 253, 0.1);
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto;
}
</style>