<template>

  <div>
    <h1>{{ className }} {{ $tr('exams') }}</h1>
    <ui-radio-group
      :name="$tr('examFilter')"
      :label="$tr('show')"
      :options="filterOptions"
      v-model="filterSelected"
      class="radio-group"
    />
    <ui-button
      type="primary"
      color="primary"
      :raised="true"
      icon="add"
      class="create-button"
      @click="openCreateExamModal">
      {{ $tr('newExam') }}
    </ui-button>
    <table v-if="sortedExams.length">
      <thead>
        <tr>
          <th class="col-icon"></th>
          <th class="col-title">{{ $tr('title') }}</th>
          <th class="col-visibility">{{ $tr('visibleTo') }}</th>
          <th class="col-action">{{ $tr('action') }}</th>
        </tr>
      </thead>
      <tbody>
        <exam-row
          v-for="exam in filteredExams"
          :key="exam.id"
          :examId="exam.id"
          :examTitle="exam.title"
          :examActive="exam.active"
          :examVisibility="exam.visibility"
          :classId="classId"
          :className="className"
          :classGroups="[]"
          @changeExamVisibility="openChangeExamVisibilityModal"
          @activateExam="openActivateExamModal"
          @deactivateExam="openDeactivateExamModal"
          @previewExam="openPreviewExamModal"
          @viewReport="routeToExamReport(exam)"
          @renameExam="openRenameExamModal"
          @deleteExam="openDeleteExamModal"
        />
      </tbody>
    </table>
    <p v-else class="center-text"><strong>{{ $tr('noExams') }}</strong></p>
    <create-exam-modal
      v-if="showCreateExamModal"
      :classId="classId"
      :channels="sortedChannels"
    />
    <activate-exam-modal
      v-if="showActivateExamModal"
      :examId="selectedExam.id"
      :examTitle="selectedExam.title"
      :examVisibility="selectedExam.visibility"
      :classId="classId"
    />
    <deactivate-exam-modal
      v-if="showDeactivateExamModal"
      :examId="selectedExam.id"
      :examTitle="selectedExam.title"
      :examVisibility="selectedExam.visibility"
      :classId="classId"
    />
    <change-exam-visibility-modal
      v-if="showChangeExamVisibilityModal"
      :examId="selectedExam.id"
      :examTitle="selectedExam.title"
      :examVisibility="selectedExam.visibility"
      :classId="classId"
      :className="className"
      :classGroups="currentClassGroups"
    />
    <preview-exam-modal
      v-if="showPreviewExamModal"
      :examChannelId="selectedExam.channelId"
      :examQuestionSources="selectedExam.questionSources"
      :examSeed="selectedExam.seed"
      :examNumQuestions="selectedExam.questionCount"
    />
    <rename-exam-modal
      v-if="showRenameExamModal"
      :examId="selectedExam.id"
      :examTitle="selectedExam.title"
      :classId="classId"
      :exams="sortedExams"
    />
    <delete-exam-modal
      v-if="showDeleteExamModal"
      :examId="selectedExam.id"
      :examTitle="selectedExam.title"
      :classId="classId"
    />
  </div>

</template>


<script>

  import { className } from '../../state/getters/main';
  import * as ExamActions from '../../state/actions/exam';
  import { Modals as ExamModals } from '../../examConstants';
  import { PageNames } from '../../constants';
  import orderBy from 'lodash/orderBy';
  import uiButton from 'keen-ui/src/UiButton';
  import uiRadioGroup from 'keen-ui/src/UiRadioGroup';
  import examRow from './exam-row';
  import createExamModal from './create-exam-modal';
  import activateExamModal from './activate-exam-modal';
  import deactivateExamModal from './deactivate-exam-modal';
  import changeExamVisibilityModal from './change-exam-visibility-modal';
  import previewExamModal from './preview-exam-modal';
  import renameExamModal from './rename-exam-modal';
  import deleteExamModal from './delete-exam-modal';
  export default {
    $trNameSpace: 'coachExamsPage',
    $trs: {
      exams: 'Exams',
      show: 'Show',
      all: 'All',
      active: 'Active',
      inactive: 'Inactive',
      examFilter: 'Exam filter',
      newExam: 'New Exam',
      title: 'Title',
      visibleTo: 'Visible to',
      action: 'Action',
      noExams: `You do not have any exams. Start by creating a new exam above.`,
    },
    components: {
      uiButton,
      uiRadioGroup,
      examRow,
      createExamModal,
      activateExamModal,
      deactivateExamModal,
      changeExamVisibilityModal,
      previewExamModal,
      renameExamModal,
      deleteExamModal,
    },
    data() {
      return {
        filterSelected: this.$tr('all'),
        selectedExam: {
          title: '',
          id: '',
          visibility: {
            class: null,
            groups: [],
          },
        },
      };
    },
    computed: {
      sortedExams() {
        return orderBy(this.exams, [exam => exam.title.toUpperCase()], ['asc']);
      },
      sortedChannels() {
        return orderBy(this.channels, [channel => channel.name.toUpperCase()], ['asc']);
      },
      filterOptions() {
        return [
          {
            label: this.$tr('all'),
            value: this.$tr('all'),
          },
          {
            label: this.$tr('active'),
            value: this.$tr('active'),
          },
          {
            label: this.$tr('inactive'),
            value: this.$tr('inactive'),
          },
        ];
      },
      activeExams() {
        return this.sortedExams.filter(exam => exam.active === true);
      },
      inactiveExams() {
        return this.sortedExams.filter(exam => exam.active === false);
      },
      filteredExams() {
        const filter = this.filterSelected;
        if (filter === this.$tr('active')) {
          return this.activeExams;
        } else if (filter === this.$tr('inactive')) {
          return this.inactiveExams;
        }
        return this.sortedExams;
      },
      showCreateExamModal() {
        return this.examModalShown === ExamModals.CREATE_EXAM;
      },
      showActivateExamModal() {
        return this.examModalShown === ExamModals.ACTIVATE_EXAM;
      },
      showDeactivateExamModal() {
        return this.examModalShown === ExamModals.DEACTIVATE_EXAM;
      },
      showChangeExamVisibilityModal() {
        return this.examModalShown === ExamModals.CHANGE_EXAM_VISIBILITY;
      },
      showPreviewExamModal() {
        return this.examModalShown === ExamModals.PREVIEW_EXAM;
      },
      showRenameExamModal() {
        return this.examModalShown === ExamModals.RENAME_EXAM;
      },
      showDeleteExamModal() {
        return this.examModalShown === ExamModals.DELETE_EXAM;
      },
    },
    methods: {
      setSelectedExam(examId) {
        Object.assign(this.selectedExam, this.sortedExams.find(exam => exam.id === examId));
      },
      openCreateExamModal() {
        this.displayExamModal(ExamModals.CREATE_EXAM);
      },
      openChangeExamVisibilityModal(examId) {
        this.setSelectedExam(examId);
        this.displayExamModal(ExamModals.CHANGE_EXAM_VISIBILITY);
      },
      openActivateExamModal(examId) {
        this.setSelectedExam(examId);
        this.displayExamModal(ExamModals.ACTIVATE_EXAM);
      },
      openDeactivateExamModal(examId) {
        this.setSelectedExam(examId);
        this.displayExamModal(ExamModals.DEACTIVATE_EXAM);
      },
      openPreviewExamModal(examId) {
        this.setSelectedExam(examId);
        this.displayExamModal(ExamModals.PREVIEW_EXAM);
      },
      routeToExamReport({ id, channelId }) {
        this.$router.push({
          name: PageNames.EXAM_REPORT,
          params: {
            classId: this.classId,
            examId: id,
            channelId,
          },
        });
      },
      openRenameExamModal(examId) {
        this.setSelectedExam(examId);
        this.displayExamModal(ExamModals.RENAME_EXAM);
      },
      openDeleteExamModal(examId) {
        this.setSelectedExam(examId);
        this.displayExamModal(ExamModals.DELETE_EXAM);
      },
    },
    vuex: {
      actions: { displayExamModal: ExamActions.displayExamModal },
      getters: {
        classId: state => state.classId,
        className,
        currentClassGroups: state => state.pageState.currentClassGroups,
        exams: state => state.pageState.exams,
        channels: state => state.pageState.channels,
        examModalShown: state => state.pageState.examModalShown,
      },
    },
  };

</script>


<style lang="stylus" scoped>

  .create-button
    float: right
    margin-top: 1em
    margin-bottom: 1em

  .center-text
    text-align: center

  .radio-group
    display: inline-block

  table
    margin-top: 3em
    width: 100%

  .col-title
    text-align: left

  .col-visibility, .col-action
    text-align: right

</style>
